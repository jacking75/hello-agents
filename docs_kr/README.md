<div align="right">
  <a href="./README_EN.md">English</a> | 中文
</div>
<div align='center'>
  <img src="./images/hello-agents.png" alt="alt text" width="100%">
  <h1>Hello-Agents</h1>
  <h3>🤖 《처음부터 시작하는 지능형 에이전트 구축》</h3>
  <div align="center">
  <a href="https://trendshift.io/repositories/15520" target="_blank">
    <img src="https://trendshift.io/api/badge/repositories/15520" alt="datawhalechina%2Fhello-agents | Trendshift" style="width: 250px; height: 55px;" width="250" height="55"/>
  </a>
  </div>
  <p><em>기초 이론부터 실제 응용까지, 지능형 에이전트 시스템의 설계와 구현을 완전히 마스터하기</em></p>
  <img src="https://img.shields.io/github/stars/datawhalechina/Hello-Agents?style=flat&logo=github" alt="GitHub stars"/>
  <img src="https://img.shields.io/github/forks/datawhalechina/Hello-Agents?style=flat&logo=github" alt="GitHub forks"/>
  <img src="https://img.shields.io/badge/language-Chinese-brightgreen?style=flat" alt="Language"/>
  <a href="https://github.com/datawhalechina/Hello-Agents"><img src="https://img.shields.io/badge/GitHub-Project-blue?style=flat&logo=github" alt="GitHub Project"></a>
  <a href="https://datawhalechina.github.io/hello-agents/"><img src="https://img.shields.io/badge/온라인%20읽기-Online%20Reading-green?style=flat&logo=gitbook" alt="온라인 읽기"></a>
</div>


---

## 🎯 프로젝트 소개

&emsp;&emsp;2024년이 "백 모델 대전"의 원년이었다면, 2025년은 의심할 여지 없이 "에이전트 원년"을 열었습니다. 기술의 초점은 더 큰 기반 모델을 학습시키는 것에서, 더 스마트한 지능형 에이전트 애플리케이션을 구축하는 것으로 이동하고 있습니다. 그러나 현재 체계적이고 실습 중심의 튜토리얼은 극도로 부족합니다. 이를 위해 Hello-Agents 프로젝트를 시작하였으며, 커뮤니티에 처음부터 이론과 실전을 겸비한 지능형 에이전트 시스템 구축 가이드를 제공하고자 합니다.

&emsp;&emsp;Hello-Agents는 Datawhale 커뮤니티의 <strong>체계적인 지능형 에이전트 학습 튜토리얼</strong>입니다. 오늘날 에이전트 구축은 크게 두 가지 유파로 나뉩니다. 하나는 Dify, Coze, n8n과 같은 소프트웨어 공학적 에이전트로, 본질적으로 프로세스 중심의 소프트웨어 개발이며 LLM이 데이터 처리 백엔드 역할을 합니다. 다른 하나는 AI 네이티브 에이전트, 즉 진정으로 AI가 주도하는 에이전트입니다. 이 튜토리얼은 후자—진정한 AI Native Agent—를 깊이 이해하고 구축하는 것을 목표로 합니다. 프레임워크의 표면을 넘어, 지능형 에이전트의 핵심 원리에서 출발하여 핵심 아키텍처를 심층적으로 이해하고, 고전적 패러다임을 파악하며, 최종적으로 직접 다중 에이전트 애플리케이션을 구축하게 됩니다. 최고의 학습 방법은 직접 실습하는 것이라 믿습니다. 이 튜토리얼이 에이전트 세계를 탐험하는 출발점이 되어, 대형 언어 모델의 "사용자"에서 지능형 에이전트 시스템의 "구축자"로 성장하는 데 도움이 되길 바랍니다.

## 🌐 온라인 읽기

**[🌐 해외 접속](https://datawhalechina.github.io/hello-agents/)** | **[🚀 국내 가속](https://hello-agents.datawhale.cc)**

### ✨ 무엇을 얻을 수 있나요?

- 📖 <strong>Datawhale 오픈소스 무료</strong> 이 프로젝트의 모든 콘텐츠를 완전 무료로 학습하고, 커뮤니티와 함께 성장
- 🔍 <strong>핵심 원리 이해</strong> 지능형 에이전트의 개념, 역사, 고전 패러다임 심층 이해
- 🏗️ <strong>직접 구현</strong> 인기 저코드 플랫폼과 에이전트 코드 프레임워크 사용법 습득
- 🛠️ <strong>자체 프레임워크 [HelloAgents](https://github.com/jjyaoao/helloagents)</strong> OpenAI 네이티브 API 기반으로 처음부터 자신만의 에이전트 프레임워크 구축
- ⚙️ <strong>고급 기술 습득</strong> 컨텍스트 엔지니어링, Memory, 프로토콜, 평가 등 시스템적 기술을 단계별로 구현
- 🤝 <strong>모델 학습</strong> Agentic RL 습득, SFT부터 GRPO까지 전 과정 실전 LLM 학습
- 🚀 <strong>실제 사례 구동</strong> 스마트 여행 어시스턴트, 사이버 타운 등 종합 프로젝트 실전 개발
- 📖 <strong>취업 면접</strong> 에이전트 취업 관련 면접 문제 학습

## 📖 목차 안내

| 챕터                                                                                   | 핵심 내용                                      | 상태 |
| -------------------------------------------------------------------------------------- | --------------------------------------------- | ---- |
| [서문](./前言.md)                                                                      | 프로젝트의 배경, 동기 및 독자 가이드           | ✅    |
| <strong>제1부: 지능형 에이전트와 언어 모델 기초</strong>                               |                                               |      |
| [제1장 에이전트 첫걸음](./chapter1/第一章%20初识智能体.md)                             | 에이전트 정의, 유형, 패러다임 및 응용          | ✅    |
| [제2장 에이전트 발전사](./chapter2/第二章%20智能体发展史.md)                           | 기호주의에서 LLM 기반 에이전트 진화까지        | ✅    |
| [제3장 대형 언어 모델 기초](./chapter3/第三章%20大语言模型基础.md)                     | Transformer, 프롬프트, 주요 LLM 및 한계       | ✅    |
| <strong>제2부: 나만의 대형 언어 모델 에이전트 구축</strong>                            |                                               |      |
| [제4장 에이전트 고전 패러다임 구축](./chapter4/第四章%20智能体经典范式构建.md)         | ReAct, Plan-and-Solve, Reflection 직접 구현   | ✅    |
| [제5장 저코드 플랫폼 기반 에이전트 구축](./chapter5/第五章%20基于低代码平台的智能体搭建.md) | Coze, Dify, n8n 등 저코드 에이전트 플랫폼 활용 | ✅    |
| [제6장 프레임워크 개발 실습](./chapter6/第六章%20框架开发实践.md)                      | AutoGen, AgentScope, LangGraph 등 주요 프레임워크 응용 | ✅    |
| [제7장 나만의 에이전트 프레임워크 구축](./chapter7/第七章%20构建你的Agent框架.md)      | 처음부터 에이전트 프레임워크 구축              | ✅    |
| <strong>제3부: 고급 지식 확장</strong>                                                 |                                               |      |
| [제8장 메모리와 검색](./chapter8/第八章%20记忆与检索.md)                               | 메모리 시스템, RAG, 저장소                    | ✅    |
| [제9장 컨텍스트 엔지니어링](./chapter9/第九章%20上下文工程.md)                        | 지속적 상호작용의 "상황 이해"                 | ✅    |
| [제10장 에이전트 통신 프로토콜](./chapter10/第十章%20智能体通信协议.md)               | MCP, A2A, ANP 등 프로토콜 분석               | ✅    |
| [제11장 Agentic-RL](./chapter11/第十一章%20Agentic-RL.md)                             | SFT부터 GRPO까지 LLM 학습 실전               | ✅    |
| [제12장 에이전트 성능 평가](./chapter12/第十二章%20智能体性能评估.md)                 | 핵심 지표, 벤치마크 및 평가 프레임워크        | ✅    |
| <strong>제4부: 종합 사례 심화</strong>                                                 |                                               |      |
| [제13장 스마트 여행 어시스턴트](./chapter13/第十三章%20智能旅行助手.md)               | MCP와 다중 에이전트 협업의 실세계 응용        | ✅    |
| [제14장 자동화 심층 연구 에이전트](./chapter14/第十四章%20自动化深度研究智能体.md)    | DeepResearch Agent 재현 및 분석              | ✅    |
| [제15장 사이버 타운 구축](./chapter15/第十五章%20构建赛博小镇.md)                     | 에이전트와 게임의 결합, 사회 역학 시뮬레이션 | ✅    |
| <strong>제5부: 졸업 프로젝트 및 미래 전망</strong>                                    |                                               |      |
| [제16장 졸업 프로젝트](./chapter16/第十六章%20毕业设计.md)                            | 완전한 다중 에이전트 애플리케이션 구축        | ✅    |

### 커뮤니티 기여 선집 (Community Blog)

&emsp;&emsp;Hello-Agents 또는 에이전트 관련 기술 학습에서 얻은 독창적인 인사이트, 실습 정리 등을 PR 형태로 커뮤니티 선집에 기여해 주세요. 본문과 독립된 내용이라면 Extra-Chapter에 투고할 수도 있습니다! <strong>첫 번째 기여를 기대합니다!</strong>

| 커뮤니티 선집                                                                                                                                      | 내용 요약                  |
| --------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------- |
| [00-공동 창작 졸업 프로젝트](https://github.com/datawhalechina/hello-agents/blob/main/Co-creation-projects)                                     | 커뮤니티 공동 창작 졸업 프로젝트 |
| [01-에이전트 면접 문제 정리](https://github.com/datawhalechina/hello-agents/blob/main/Extra-Chapter/Extra01-面试问题总结.md)                      | 에이전트 직무 관련 면접 문제 |
| [01-에이전트 면접 문제 답안](https://github.com/datawhalechina/hello-agents/blob/main/Extra-Chapter/Extra01-参考答案.md)                          | 관련 면접 문제 답안 |
| [02-컨텍스트 엔지니어링 내용 보충](https://github.com/datawhalechina/hello-agents/blob/main/Extra-Chapter/Extra02-上下文工程补充知识.md)           | 컨텍스트 엔지니어링 내용 확장 |
| [03-Dify 에이전트 생성 단계별 튜토리얼](https://github.com/datawhalechina/hello-agents/blob/main/Extra-Chapter/Extra03-Dify智能体创建保姆级操作流程.md) | Dify 에이전트 생성 상세 튜토리얼 |
| [04-Hello-agents 강좌 자주 묻는 질문](https://github.com/datawhalechina/hello-agents/blob/main/Extra-Chapter/Extra04-DatawhaleFAQ.md)            | Datawhale 강좌 자주 묻는 질문 |
| [05-Agent Skills와 MCP 비교 해설](https://github.com/datawhalechina/hello-agents/blob/main/Extra-Chapter/Extra05-AgentSkills解读.md)             | Agent Skills와 MCP 기술 비교 |
| [06-GUI Agent 개론과 실전](https://github.com/datawhalechina/hello-agents/blob/main/Extra-Chapter/Extra06-GUIAgent科普与实战.md)                  | GUI Agent 개론과 다양한 시나리오 실전 |
| [07-환경 설정](https://github.com/datawhalechina/hello-agents/blob/main/Extra-Chapter/Extra07-环境配置.md)                                       | 환경 설정 |
| [08-좋은 Skill 작성법](https://github.com/datawhalechina/hello-agents/blob/main/Extra-Chapter/Extra08-如何写出好的Skill.md)                      | Skill 작성 모범 사례 |
| [09-에이전트 애플리케이션 개발 실전 경험 공유](https://github.com/datawhalechina/hello-agents/blob/main/Extra-Chapter/Extra09-Agent应用开发实践踩坑与经验分享.md) | Code Agent 애플리케이션 개발 경험 및 교훈 |

### PDF 버전 다운로드

&emsp;&emsp; *<strong>이 Hello-Agents PDF 튜토리얼은 완전히 오픈소스이며 무료입니다. 각종 마케팅 계정이 워터마크를 추가해 다중 에이전트 시스템 입문자에게 판매하는 것을 방지하기 위해, PDF 파일에 미리 읽기에 영향을 주지 않는 Datawhale 오픈소스 로고 워터마크를 추가하였으니 양해 부탁드립니다～</strong>*

> *Hello-Agents PDF : https://github.com/datawhalechina/hello-agents/releases/tag/V1.0.0*  
> *Hello-Agents PDF 국내 다운로드 주소 : https://www.datawhale.cn/learn/summary/239* 

## 💡 학습 방법

&emsp;&emsp;미래의 지능형 시스템 구축자 여러분을 환영합니다! 이 흥미진진한 여정을 시작하기 전에, 몇 가지 명확한 안내를 드리겠습니다.

&emsp;&emsp;이 프로젝트의 내용은 이론과 실전을 균형 있게 다루며, 단일 에이전트에서 다중 에이전트 시스템의 설계 및 개발 전 과정을 체계적으로 습득하는 데 도움을 줍니다. 따라서 어느 정도 프로그래밍 기초가 있는 <strong>AI 개발자, 소프트웨어 엔지니어, 재학생</strong>과 최신 AI 기술에 강한 관심을 가진 <strong>독학자</strong>에게 특히 적합합니다. 이 프로젝트를 학습하기 전에 기초적인 Python 프로그래밍 능력과 대형 언어 모델에 대한 기본적인 개념적 이해(예: API를 통해 LLM을 호출하는 방법)를 갖추고 있기를 권장합니다. 프로젝트의 초점은 응용과 구축이므로, 깊은 알고리즘 또는 모델 학습 배경은 필요하지 않습니다.

&emsp;&emsp;프로젝트는 다섯 개의 큰 부분으로 나뉘며, 각 부분은 다음 단계로 나아가는 튼튼한 디딤돌입니다:

- <strong>제1부: 지능형 에이전트와 언어 모델 기초</strong>（제1장~제3장）, 에이전트의 정의, 유형, 발전 역사부터 시작하여 "에이전트"라는 개념의 유래와 맥락을 정리합니다. 이후 대형 언어 모델의 핵심 지식을 빠르게 다지며, 실습 여정을 위한 탄탄한 이론적 기반을 마련합니다.

- <strong>제2부: 나만의 대형 언어 모델 에이전트 구축</strong>（제4장~제7장）, 직접 실습의 출발점입니다. ReAct 등 고전 패러다임을 직접 구현하고, Coze 등 저코드 플랫폼의 편리함을 경험하며, LangGraph 등 주요 프레임워크의 응용을 습득합니다. 최종적으로 처음부터 자신만의 에이전트 프레임워크를 구축하여, "바퀴를 사용하는" 능력과 "바퀴를 만드는" 능력을 모두 갖추게 됩니다.

- <strong>제3부: 고급 지식 확장</strong>（제8장~제12장）, 이 부분에서 여러분의 에이전트는 "생각하고 협력하는 법"을 배우게 됩니다. 제2부의 자체 개발 프레임워크를 활용하여 메모리와 검색, 컨텍스트 엔지니어링, 에이전트 학습 등 핵심 기술을 심층 탐구하고, 다중 에이전트 간 통신 프로토콜을 학습합니다. 최종적으로 에이전트 시스템 성능을 전문적으로 평가하는 방법을 습득합니다.

- <strong>제4부: 종합 사례 심화</strong>（제13장~제15장）, 이론과 실습의 교차점입니다. 배운 내용을 통합하여 스마트 여행 어시스턴트, 자동화 심층 연구 에이전트, 심지어 사회 역학을 시뮬레이션하는 사이버 타운을 직접 만들며, 실제적이고 흥미로운 프로젝트에서 구축 능력을 단련합니다.

- <strong>제5부: 졸업 프로젝트 및 미래 전망</strong>（제16장）, 여정의 끝에서 졸업 프로젝트를 맞이하게 됩니다. 완전한, 자신만의 다중 에이전트 애플리케이션을 구축하며 학습 성과를 전면적으로 검증합니다. 또한 에이전트의 미래를 함께 전망하며 흥미진진한 최신 방향을 탐색합니다.


&emsp;&emsp;에이전트는 빠르게 발전하고 실습에 크게 의존하는 분야입니다. 최적의 학습 효과를 위해 프로젝트의 `code` 폴더에 모든 배포 코드를 제공하였으며, <strong>이론과 실습을 결합</strong>할 것을 강력히 권장합니다. 프로젝트에서 제공하는 모든 코드를 직접 실행하고, 디버깅하고, 심지어 수정해 보세요. 언제든지 Datawhale 및 다른 에이전트 관련 커뮤니티를 팔로우하고, 문제가 생기면 이 프로젝트의 issue 섹션에서 질문할 수 있습니다.

&emsp;&emsp;자, 에이전트의 신비로운 세계로 들어갈 준비가 되셨나요? 지금 바로 출발합시다!

## 🤝 기여 방법

저희는 개방적인 오픈소스 커뮤니티로, 어떠한 형태의 기여도 환영합니다!

- 🐛 <strong>버그 신고</strong> - 내용 또는 코드 문제 발견 시 Issue 제출
- 💡 <strong>제안 제출</strong> - 프로젝트에 좋은 아이디어가 있다면 토론을 시작하세요
- 📝 <strong>내용 개선</strong> - 튜토리얼 개선을 도와 Pull Request 제출
- ✍️ <strong>실습 공유</strong> - "커뮤니티 기여 선집"에서 학습 노트와 프로젝트 공유

## 🙏 감사의 말

### 핵심 기여자
- [천쓰저우-프로젝트 책임자](https://github.com/jjyaoao) (Datawhale 멤버, 전체 집필 및 교정)
- [쑨타오-공동 발기인](https://github.com/fengju0213) (Datawhale 멤버、CAMEL-AI, 제9장 내용 및 교정)
- [장수판-공동 발기인](https://github.com/Tsumugii24)（Datawhale 멤버, 챕터 연습 문제 설계 및 교정）
- [황페이린-Datawhale 의향 멤버](https://github.com/HeteroCat) (에이전트 개발 엔지니어, 제5장 내용 기여자)
- [쩡신민-에이전트 엔지니어](https://github.com/fancyboi999) (뉴커 테크놀로지, 제14장 사례 개발)
- [주신중-지도 전문가](https://xinzhongzhu.github.io/) (Datawhale 수석 과학자-저장사범대학교 항저우 인공지능연구원 교수)

### Extra-Chapter 기여자
- [WH](https://github.com/WHQAQ11) (내용 기여자)
- [저우아오제-DW 기여자팀](https://github.com/thunderbolt-fire) (시안교통대학교, Extra02 내용 기여)
- [장천쉬-개인 개발자](https://github.com/Tasselszcx)(임페리얼 칼리지 런던, Extra03 내용 기여)
- [황홍한-DW 기여자팀](https://github.com/XiaoMa-PM) (선전대학교, Extra04 내용 기여)
- [왕다펑-Datawhale 멤버](https://github.com/ditingdapeng) (시니어 개발 엔지니어, Extra08 내용 기여)
- [유이후이-개인 개발자](https://github.com/YYHDBL) (난징정보공정대학교, Extra09 내용 기여)

### 특별 감사
- [@Sm1les](https://github.com/Sm1les)의 이 프로젝트에 대한 도움과 지원에 감사드립니다
- 이 프로젝트에 기여하신 모든 개발자분들께 감사드립니다 ❤️

<div align=center style="margin-top: 30px;">
  <a href="https://github.com/datawhalechina/Hello-Agents/graphs/contributors">
    <img src="https://contrib.rocks/image?repo=datawhalechina/Hello-Agents" />
  </a>
</div>

## Star 히스토리

<div align='center'>
    <img src="./images/star-history-2026324.png" alt="Datawhale" width="90%">
</div>

<div align="center">
  <p>⭐ 이 프로젝트가 도움이 되었다면, Star를 눌러 주세요!</p>
</div>

## 독자 교류 그룹

<div align='center'>
    <img src="./读者群二维码.png" alt="독자 그룹 QR코드" width="30%">
    <p>QR코드를 스캔하여 독자 교류 그룹에 참여하고, 더 많은 학습자들과 소통하세요</p>
</div>

## Datawhale 소개

<div align='center'>
    <img src="./images/datawhale.png" alt="Datawhale" width="30%">
    <p>QR코드를 스캔하여 Datawhale 공식 계정을 팔로우하고, 더 많은 우수한 오픈소스 콘텐츠를 받아보세요</p>
</div>

---

## 📜 오픈소스 라이선스

본 저작물은 [크리에이티브 커먼즈 저작자표시-비영리-동일조건변경허락 4.0 국제 라이선스](http://creativecommons.org/licenses/by-nc-sa/4.0/)에 따라 이용할 수 있습니다.
