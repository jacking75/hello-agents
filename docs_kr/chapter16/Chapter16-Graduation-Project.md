# 제16장: 졸업 프로젝트 - 나만의 멀티 에이전트 애플리케이션 만들기

Hello-Agents 튜토리얼의 마지막 장에 도달하신 것을 축하합니다! 이전 15개 챕터에서 우리는 HelloAgents 프레임워크를 처음부터 구축하고, 에이전트의 핵심 개념, 다양한 패러다임, 도구 시스템, 메모리 메커니즘, 통신 프로토콜, 강화 학습 훈련, 성능 평가에 대해 학습했습니다. 13~15장에서는 세 가지 완전한 실습 프로젝트(지능형 여행 보조자, 자동화된 DeepResearch 에이전트, 사이버 타운)를 통해 학습한 모든 지식을 통합하는 방법을 시연했습니다.

이제 여러분이 진정한 에이전트 시스템 개발자가 될 시간입니다! 이 장에서는 **나만의 멀티 에이전트 애플리케이션을 구축**하고 오픈소스 협업을 통해 커뮤니티와 성과를 공유하는 방법을 안내합니다.

## 16.1 졸업 프로젝트의 의미

### 16.1.1 왜 졸업 프로젝트를 하는가

기술을 배우는 가장 좋은 방법은 튜토리얼을 읽는 것이 아니라 **직접 실습**하는 것입니다. 이전 챕터들을 통해 에이전트 시스템 구축에 관한 이론적 지식과 기술 도구를 습득했습니다. 그러나 진정한 도전은 다음과 같습니다: **이 지식을 실제 문제에 어떻게 적용할 것인가? 완전한 시스템을 어떻게 설계할 것인가? 다양한 엣지 케이스와 예외를 어떻게 처리할 것인가?**

졸업 프로젝트의 핵심 가치는 종합적인 응용 능력을 기르는 것입니다. 이전에 학습한 모든 지식(에이전트 패러다임, 도구 시스템, 메모리 메커니즘, 통신 프로토콜 등)을 선택적으로 통합하여 완전한 프로젝트로 만드는 것입니다.

이 장의 학습과 실습을 통해 여러분이 완전한 에이전트 애플리케이션을 독립적으로 설계하고 구현하고, HelloAgents 프레임워크의 다양한 기능을 능숙하게 사용하고, 기본적인 Git 및 GitHub 작업을 숙달하고, 명확한 프로젝트 문서 작성 방법을 배우고, 오픈소스 커뮤니티 협업 개발에 참여하고, 궁극적으로 공개적으로 선보일 수 있는 기술적 결과물을 얻을 수 있기를 바랍니다.

### 16.1.2 졸업 프로젝트의 형태

졸업 프로젝트는 Hello-Agents 공동 창작 프로젝트 저장소(`Co-creation-projects` 디렉토리)에 **오픈소스 프로젝트** 형태로 제출됩니다. 구체적인 요구사항은 다음과 같습니다:

1. **프로젝트 이름**: `{GitHub-사용자명}-{프로젝트명}` 형식 사용, 예: `jjyaoao-CodeReviewAgent`

2. **프로젝트 내용**:
   - 실행 가능한 Jupyter Notebook (`.ipynb` 파일) 또는 Python 스크립트
   - 완전한 의존성 목록 (`requirements.txt`)
   - 명확한 README 문서 (`README.md`)
   - 선택 사항: 데모 동영상, 스크린샷, 데이터셋 등

3. **제출 방법**: GitHub Pull Request (PR)를 통해 제출

4. **검토 프로세스**: 커뮤니티 구성원이 코드를 검토하고 개선 제안을 제공하며, 승인 후 메인 저장소에 병합

## 16.2 프로젝트 주제 선정 가이드

### 16.2.1 주제 선정 원칙

좋은 졸업 프로젝트는 실용적이어야 합니다. 기술을 위한 기술이 아닌 실제 문제를 해결해야 합니다. 제한된 시간과 자원 내에서 완성도를 추구하는 동시에 여러분의 기술 역량을 명확하게 보여주어야 합니다.

### 16.2.2 추천 주제 방향

다음은 추천 프로젝트 방향입니다. 하나를 선택하거나 자신만의 아이디어를 제안할 수 있습니다:

**(1) 생산성 도구**

- **지능형 코드 리뷰 보조자**: 코드 품질을 자동으로 분석하고 잠재적 버그를 발견하며 최적화 제안 제공
- **지능형 문서 생성기**: 코드를 기반으로 API 문서 및 사용자 매뉴얼을 자동으로 생성
- **지능형 회의 보조자**: 회의 내용을 기록하고, 회의록을 생성하고, 실행 항목을 추출
- **지능형 이메일 보조자**: 이메일을 자동으로 분류하고, 답장 초안을 생성하고, 중요한 사항을 알림

**(2) 학습 지원**

- **지능형 학습 파트너**: 학습 진도에 따라 학습 자료를 추천하고, 연습 문제를 생성하고, 질문에 답변
- **지능형 논문 보조자**: 문헌 찾기, 논문 요약, 인용 생성 지원
- **지능형 프로그래밍 튜터**: 프로그래밍 연습 제공, 코드 리뷰, 학습 경로 계획
- **지능형 언어 학습 보조자**: 대화 연습, 문법 교정, 어휘 확장 제공

**(3) 창의적 엔터테인먼트**

- **지능형 스토리 생성기**: 사용자 입력을 기반으로 소설, 대본, 시 생성
- **지능형 게임 NPC**: 개성 있는 게임 캐릭터를 만들어 플레이어와 자연스럽게 대화
- **지능형 음악 추천**: 기분과 장면에 따라 음악을 추천하고 재생 목록 생성
- **지능형 레시피 보조자**: 재료와 취향에 따라 레시피를 추천하고 쇼핑 목록 생성

**(4) 데이터 분석**

- **지능형 데이터 분석가**: 데이터를 자동으로 분석하고 시각화 차트를 생성하고 분석 보고서 작성
- **지능형 주식 분석**: 주식 데이터와 뉴스 감성을 분석하고 투자 조언 제공
- **지능형 여론 모니터링**: 소셜 미디어와 뉴스 사이트를 모니터링하고 여론 동향 분석
- **지능형 경쟁사 분석**: 경쟁사 정보를 수집하고 비교 분석하여 보고서 생성

**(5) 생활 서비스**

- **지능형 건강 보조자**: 건강 데이터를 기록하고 건강 조언을 제공하며 운동 계획 수립
- **지능형 재무 보조자**: 수입과 지출을 기록하고 소비 습관을 분석하며 재무 조언 제공
- **지능형 쇼핑 보조자**: 가격을 비교하고 제품을 추천하며 쇼핑 목록 생성
- **지능형 홈 컨트롤**: 자연어를 통해 스마트 홈 기기를 제어

### 16.2.3 주제 선정 예시

구체적인 예시를 통해 주제를 선정하고 프로젝트를 설계하는 방법을 설명하겠습니다.

**프로젝트명**: 지능형 코드 리뷰 보조자 (CodeReviewAgent)

**문제 분석**: 코드 리뷰는 소프트웨어 개발의 중요한 부분이지만 수동 리뷰는 시간이 많이 걸리고 문제를 놓치기 쉽습니다. 기존의 정적 분석 도구는 구문 오류만 찾을 수 있고 코드 로직을 이해할 수 없으므로, 코드 의미를 이해하고 심층 분석을 제공할 수 있는 지능형 보조자가 필요합니다.

**핵심 기능**: 이 프로젝트는 코드 품질 분석(코드 스타일, 명명 규칙, 주석 완성도 확인), 잠재적 버그 감지(로직 오류, 경계 조건 문제, 리소스 누수 발견), 성능 최적화 제안(성능 병목 현상 파악, 최적화 방안 제안), 보안 취약점 스캔(SQL 인젝션, XSS 및 기타 보안 문제 탐지), 모범 사례 권장(언어 기능과 디자인 패턴에 따른 개선 사항 제안)을 구현할 것입니다.

**예상 결과물**: 최종 결과물은 전체 리뷰 과정을 시연하는 실행 가능한 Jupyter Notebook이 될 것이며, Python 및 JavaScript와 같은 주류 언어를 지원하고, 구조화된 Markdown 형식의 리뷰 보고서를 생성하며, 구체적인 코드 예시와 개선 제안을 제공합니다.

## 16.3 개발 환경 준비

### 16.3.1 필요한 도구 설치

개발을 시작하기 전에 개발 환경에 다음 도구가 설치되어 있는지 확인하십시오:

**(1) Python 환경**

```bash
# HelloAgents 설치
pip install "hello-agents[all]"
```

**(2) Git 및 GitHub**

```bash
# Git 버전 확인
git --version

# Git 사용자 정보 구성
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# GitHub SSH 키 구성 (권장)
# 1. SSH 키 생성
ssh-keygen -t ed25519 -C "your.email@example.com"

# 2. 공개 키를 GitHub에 추가
# ~/.ssh/id_ed25519.pub의 내용을 복사
# GitHub Settings > SSH and GPG keys에서 추가

# 3. 연결 테스트
ssh -T git@github.com
```

**(3) Jupyter Notebook**

```bash
# Jupyter 설치
pip install jupyter notebook

# 또는 JupyterLab 사용 (권장)
pip install jupyterlab

# Jupyter 시작
jupyter lab
```

### 16.3.2 프로젝트 저장소 Fork

**1단계: 저장소 Fork**

1. Hello-Agents 저장소 방문: https://github.com/datawhalechina/hello-agents
2. 오른쪽 상단의 "Fork" 버튼 클릭 (그림 16.1의 빨간 상자 참조)
3. GitHub 계정을 선택하여 Fork 생성

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/hello-agents/main/docs/images/16-figures/16-1.png" alt="" width="85%"/>
  <p>그림 16.1 저장소 Fork 단계</p>
</div>

**2단계: 로컬에 복제**

```bash
# 그림 16.2와 같이 Fork된 저장소를 복제
git clone git@github.com:your-username/hello-agents.git

# 프로젝트 디렉토리로 이동
cd Hello-Agents

# 업스트림 저장소 추가 (업데이트 동기화용)
git remote add upstream https://github.com/datawhalechina/hello-agents.git

# 원격 저장소 보기
git remote -v
```

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/hello-agents/main/docs/images/16-figures/16-2.png" alt="" width="85%"/>
  <p>그림 16.2 저장소를 로컬에 복제</p>
</div>

**3단계: 개발 브랜치 생성**

```bash
# 새 브랜치 생성 및 전환
git checkout -b feature/your-project-name

# 예:
git checkout -b feature/code-review-agent
```

### 16.3.3 프로젝트 디렉토리 구조

`Co-creation-projects` 디렉토리에 프로젝트 폴더를 생성합니다:

```bash
# 공동 창작 프로젝트 디렉토리로 이동
cd Co-creation-projects

# 프로젝트 폴더 생성 (형식: GitHub-사용자명-프로젝트명)
mkdir your-username-project-name

# 예:
mkdir jjyaoao-CodeReviewAgent

# 프로젝트 디렉토리로 이동
cd jjyaoao-CodeReviewAgent
```

추천 프로젝트 구조:

```
jjyaoao-CodeReviewAgent/
├── README.md              # 프로젝트 문서
├── requirements.txt       # Python 의존성 목록
├── main.ipynb            # 메인 Jupyter Notebook
├── data/                 # 데이터 파일 (선택 사항)
│   ├── sample_code.py
│   └── test_cases.json
├── outputs/              # 출력 결과 (선택 사항)
│   ├── review_report.md
│   └── screenshots/
├── src/                  # 소스 코드 (선택 사항, 코드가 방대한 경우)
│   ├── agents/
│   ├── tools/
│   └── utils/
└── .env.example          # 환경 변수 템플릿
```

## 16.4 프로젝트 개발 가이드

### 16.4.1 README 문서 작성

README는 프로젝트의 얼굴입니다. 좋은 README는 다음 내용을 포함해야 합니다:

```markdown
# 프로젝트명

> 프로젝트에 대한 한 문장 설명

## 📝 프로젝트 소개

프로젝트에 대한 상세한 소개:
- 어떤 문제를 해결하는가?
- 어떤 특별한 기능이 있는가?
- 어떤 시나리오에 적합한가?

## ✨ 핵심 기능

- [ ] 기능 1: 설명
- [ ] 기능 2: 설명
- [ ] 기능 3: 설명

## 🛠️ 기술 스택

- HelloAgents 프레임워크
- 사용된 에이전트 패러다임 (예: ReAct, Plan-and-Solve 등)
- 사용된 도구 및 API
- 기타 의존성 라이브러리

## 🚀 빠른 시작

### 환경 요구사항

- Python 3.10+
- 기타 요구사항

### 의존성 설치


pip install -r requirements.txt


### API 키 구성


# .env 파일 생성
cp .env.example .env

# .env 파일을 편집하여 API 키 입력


### 프로젝트 실행


# Jupyter Notebook 시작
jupyter lab

# main.ipynb를 열고 실행


## 📖 사용 예시

프로젝트 사용 방법을 보여주세요. 가급적 코드 예시와 결과를 포함하세요.

## 🎯 프로젝트 하이라이트

- 하이라이트 1: 설명
- 하이라이트 2: 설명
- 하이라이트 3: 설명

## 📊 성능 평가

평가 결과가 있는 경우 여기에 표시하세요:
- 정확도: XX%
- 응답 시간: XX초
- 기타 지표

## 🔮 향후 계획

- [ ] 구현 예정 기능 1
- [ ] 구현 예정 기능 2
- [ ] 최적화 예정 부분

## 🤝 기여 가이드라인

Issues와 Pull Request를 환영합니다!

## 📄 라이선스

MIT License

## 👤 작성자

- GitHub: [@your-username](https://github.com/your-username)
- 이메일: your.email@example.com (선택 사항)

## 🙏 감사의 말

Datawhale 커뮤니티와 Hello-Agents 프로젝트에 감사드립니다!
```

### 16.4.2 requirements.txt 작성

프로젝트에 필요한 모든 Python 의존성을 나열합니다:

```txt
# 핵심 의존성
hello-agents[all]>=0.2.7

# 시각화 (필요한 경우)
matplotlib>=3.7.0
plotly>=5.14.0

# 웹 프레임워크 (필요한 경우)
fastapi>=0.109.0
uvicorn>=0.27.0
```

### 16.4.3 Jupyter Notebook 개발

**(1) Notebook 구조 권장사항**

좋은 Jupyter Notebook은 다음 부분을 포함해야 합니다:

```python
# ========================================
# 1부: 프로젝트 소개
# ========================================

"""
# 프로젝트명

## 프로젝트 소개
프로젝트 목표와 기능에 대한 간단한 소개

## 작성자 정보
- 이름: XXX
- GitHub: @XXX
- 날짜: 2025-XX-XX
"""

# ========================================
# 2부: 환경 구성
# ========================================

# 의존성 설치
!pip install -q hello-agents[all]

# 필요한 라이브러리 가져오기
from hello_agents import SimpleAgent, HelloAgentsLLM
from hello_agents.tools import BaseTool
import os
from dotenv import load_dotenv

# 환경 변수 로드
load_dotenv()

# ========================================
# 3부: 도구 정의
# ========================================

class CustomTool(BaseTool):
    """커스텀 도구 클래스"""

    name = "tool_name"
    description = "도구 설명"

    def run(self, query: str) -> str:
        """도구 실행 로직"""
        # 도구 로직 구현
        return "결과"

# ========================================
# 4부: 에이전트 구성
# ========================================

# LLM 생성
llm = HelloAgentsLLM()

# 에이전트 생성
agent = SimpleAgent(
    name="에이전트 이름",
    llm=llm,
    system_prompt="시스템 프롬프트"
)

# 도구 추가
agent.add_tool(CustomTool())

# ========================================
# 5부: 기능 시연
# ========================================

# 예시 1: 기본 기능
print("=== 예시 1: 기본 기능 ===")
result = agent.run("사용자 입력")
print(result)

# 예시 2: 복잡한 시나리오
print("\n=== 예시 2: 복잡한 시나리오 ===")
result = agent.run("복잡한 사용자 입력")
print(result)

# ========================================
# 6부: 성능 평가 (선택 사항)
# ========================================

# 평가 코드
# ...

# ========================================
# 7부: 요약 및 전망
# ========================================

"""
## 프로젝트 요약

### 구현된 기능
- 기능 1
- 기능 2

### 직면한 도전과 해결책
- 도전 1과 해결책
- 도전 2와 해결책

### 향후 개선 방향
- 개선 사항 1
- 개선 사항 2
"""
```

### 16.4.4 프로젝트 테스트

제출 전에 이 체크리스트를 사용하여 프로젝트가 제출 요구사항을 충족하는지 확인하세요:

```markdown
- [ ] 코드가 오류 없이 정상 실행됨
- [ ] README 문서가 완전하고 명확한 지침 포함
- [ ] requirements.txt에 모든 의존성 포함
- [ ] 명확한 사용 예시 제공
- [ ] 코드에 적절한 주석 있음
- [ ] 출력 결과가 기대치에 맞음
- [ ] 일반적인 예외 케이스 처리됨
- [ ] 프로젝트 구조가 명확하고 파일 이름이 표준화됨
- [ ] 대용량 파일이 적절하게 처리됨 (다음 섹션 참조)
```

### 16.4.5 대용량 파일 처리 가이드

**⚠️ 중요: 메인 저장소가 과도하게 커지는 것을 방지하세요**

Hello-Agents 메인 저장소를 가볍게 유지하기 위해 다음 대용량 파일 처리 가이드라인을 따르세요:

**(1) 파일 크기 제한**

- **전체 프로젝트 크기**: 5MB를 초과하지 않음
- **직접 제출 금지**: 동영상 파일, 대용량 데이터셋, 모델 파일

**(2) 대용량 파일 처리 방법**

프로젝트에 대용량 파일(데이터셋, 동영상, 모델 등)이 포함된 경우 다음 방법을 사용하세요:

**방법 1: 외부 링크 사용 (권장)**

대용량 파일을 외부 플랫폼에 업로드하고 README에 다운로드 링크를 제공합니다:

```markdown
## 데이터셋

이 프로젝트에서 사용하는 데이터셋은 용량이 큽니다. 다음 링크에서 다운로드하세요:

- 데이터셋 1: [Baidu Netdisk](링크) 추출 코드: xxxx
- 데이터셋 2: [Google Drive](링크)
- 데모 동영상: [Bilibili](링크) / [YouTube](링크)
```

추천 외부 플랫폼:
- **데이터셋**: Baidu Netdisk, Google Drive, Kaggle, HuggingFace Datasets
- **동영상**: Bilibili, YouTube, Tencent Video
- **모델**: HuggingFace Models, ModelScope
- **이미지**: GitHub Issues, 이미지 호스팅 서비스

**방법 2: 독립 저장소 생성**

프로젝트에 많은 리소스가 있는 경우 독립 데이터 저장소 생성을 고려하세요:

```markdown
## 프로젝트 리소스

데이터와 데모 리소스가 많아 별도의 리소스 저장소를 만들었습니다:

- 리소스 저장소: https://github.com/your-username/project-name-resources
- 포함: 데이터셋, 데모 동영상, 모델 파일, 테스트 데이터 등

### 사용 방법

\`\`\`bash
# 리소스 저장소 복제
git clone https://github.com/your-username/project-name-resources.git

# 데이터를 프로젝트 디렉토리에 복사
cp -r project-name-resources/data ./data
\`\`\`
```

**방법 3: 샘플 데이터 사용**

메인 저장소에는 소규모 샘플 데이터만 제공합니다:

```python
# README에 설명
## 데이터 설명

- `data/sample.csv`: 샘플 데이터 (100개 레코드)
- 전체 데이터셋 (10만 개 레코드)는 [여기](링크)에서 다운로드
```

**(3) 모범 사례 예시**

```
your-username-project-name/
├── README.md              # 외부 리소스 링크 포함
├── requirements.txt
├── main.ipynb
├── .gitignore            # 대용량 파일 무시
├── data/
│   └── sample.csv        # 샘플 데이터만 (<1MB)
└── outputs/
    └── demo_result.png   # 데모 결과만 (<1MB)
```

README 설명:

```markdown
## 데이터 및 리소스

### 샘플 데이터
프로젝트에는 빠른 테스트를 위한 소규모 샘플 데이터가 포함되어 있습니다 (`data/sample.csv`에 위치)

### 전체 데이터셋
전체 데이터셋 (500MB)은 다음 링크에서 다운로드하세요:
- Baidu Netdisk: [링크] 추출 코드: xxxx
- 다운로드 후 `data/` 디렉토리에 압축 해제

### 데모 동영상
- Bilibili: [프로젝트 데모 동영상](링크)
- YouTube: [데모 동영상](링크)
```

## 16.5 Pull Request 제출

### 16.5.1 GitHub에 코드 제출

**1단계: 수정 사항 확인**

```bash
# 수정된 파일 보기
git status
```

**2단계: 파일 추가**

```bash
# 모든 수정된 파일 추가
git add .

# 또는 특정 파일 추가
git add Co-creation-projects/your-username-project-name/
```

**3단계: 변경 사항 커밋**

커밋 메시지는 다음 형식을 따라야 합니다:

```bash
# 형식: 유형: 간략한 설명
git commit -m "feat: XXX 졸업 프로젝트 추가"
```

**커밋 유형 사양:**

- `feat`: 새로운 기능 또는 프로젝트 (졸업 프로젝트에 이 유형 사용)
- `fix`: 버그 수정
- `docs`: 문서 업데이트
- `style`: 코드 형식 조정 (기능에 영향 없음)
- `refactor`: 코드 리팩토링
- `test`: 테스트 관련
- `chore`: 기타 수정 (예: 의존성 업데이트)

**4단계: GitHub에 푸시**

```bash
# Fork된 저장소에 푸시
git push origin feature/your-project-name
```

### 16.5.2 Pull Request 생성

**1단계: GitHub 방문**

1. Fork된 저장소 방문: `https://github.com/your-username/hello-agents`
2. "Pull requests" 탭 클릭 (그림 16.3 참조)
3. "New pull request" 버튼 클릭

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/hello-agents/main/docs/images/16-figures/16-3.png" alt="" width="85%"/>
  <p>그림 16.3 Pull Request 생성</p>
</div>

**2단계: 브랜치 선택**

- 기본 저장소: `datawhalechina/hello-agents`
- 기본 브랜치: `main`
- 헤드 저장소: `your-username/hello-agents`
- 비교 브랜치: `feature/your-project-name`

**3단계: PR 정보 입력**

**⚠️ 중요: 통일된 PR 제목 형식**

관리와 검색을 용이하게 하기 위해 모든 졸업 프로젝트 PR 제목은 다음 형식을 따라야 합니다:

```
[졸업 프로젝트] 프로젝트명 - 간략한 설명
```

예시:
- `[졸업 프로젝트] CodeReviewAgent - 지능형 코드 리뷰 보조자`
- `[졸업 프로젝트] StudyBuddy - AI 학습 파트너`
- `[졸업 프로젝트] DataAnalyst - 지능형 데이터 분석가`

**PR 설명 템플릿:**

```markdown
## 프로젝트 정보

- **프로젝트명**: XXX
- **작성자**: @your-username
- **프로젝트 유형**: 생산성 도구/학습 지원/창의적 엔터테인먼트/데이터 분석/생활 서비스

## 프로젝트 소개

프로젝트에 대한 간략한 설명 (2-3문장)

## 핵심 기능

- [ ] 기능 1
- [ ] 기능 2
- [ ] 기능 3

## 기술적 하이라이트

- XXX 패러다임 사용
- XXX 기능 구현
- XXX 성능 최적화

## 데모 효과

(선택 사항) 프로젝트 효과를 보여주는 스크린샷 또는 GIF 추가

## 자체 점검 목록

- [ ] 코드가 정상 실행됨
- [ ] README 문서 완전함
- [ ] requirements.txt 완전함
- [ ] 명확한 사용 예시 제공
- [ ] 코드에 적절한 주석 있음

## 기타 사항

(선택 사항) 설명이 필요한 기타 내용
```

**4단계: PR 제출**

그림 16.4와 같이 "Create pull request" 버튼을 클릭하여 제출합니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/hello-agents/main/docs/images/16-figures/16-4.png" alt="" width="85%"/>
  <p>그림 16.4 Pull Request 제출</p>
</div>

### 16.5.3 리뷰 댓글에 응답

PR을 제출한 후 커뮤니티 구성원이 코드를 검토하고 제안을 제공합니다. 신속하게 응답하세요:

1. **댓글 보기**: PR 페이지에서 리뷰어의 댓글 확인
2. **코드 수정**: 제안에 따라 코드 수정
3. **업데이트 제출**:
   ```bash
   git add .
   git commit -m "fix: 리뷰 댓글에 따라 XXX 수정"
   git push origin feature/your-project-name
   ```
4. **댓글에 답변**: GitHub에서 리뷰어에게 답변하고 수정 사항 설명

## 16.6 예시 프로젝트 소개

졸업 프로젝트 요구사항을 더 잘 이해할 수 있도록 완전한 예시 프로젝트를 소개합니다. 걱정하지 마세요 - 작은 창의적인 아이디어도 포함될 수 있습니다. 직접 만든 모든 작품은 소중합니다.

**프로젝트 정보**

- **프로젝트명**: CodeReviewAgent
- **작성자**: @jjyaoao
- **프로젝트 경로**: `Co-creation-projects/jjyaoao-CodeReviewAgent/`

**프로젝트 구조**

```
jjyaoao-CodeReviewAgent/
├── README.md              # 프로젝트 문서
├── requirements.txt       # 의존성 목록
├── main.ipynb            # 메인 프로그램 (빠른 데모와 전체 기능 포함)
├── .env.example          # 환경 변수 예시
├── .gitignore            # Git 무시 규칙
├── data/
│   └── sample_code.py    # 샘플 코드
└── outputs/
    └── review_report.md  # 샘플 보고서
```

**핵심 코드 스니펫 (main.ipynb)**

```python
# ========================================
# 지능형 코드 리뷰 보조자
# ========================================

from hello_agents import SimpleAgent, HelloAgentsLLM, ToolRegistry
from hello_agents.tools import Tool, ToolParameter
from typing import Dict, Any, List
import ast
import os

# ========================================
# 0. LLM 파라미터 구성
# ========================================

os.environ["LLM_MODEL_ID"] = "Qwen/Qwen2.5-72B-Instruct"
os.environ["LLM_API_KEY"] = "your_api_key_here"
os.environ["LLM_BASE_URL"] = "https://api-inference.modelscope.cn/v1/"
os.environ["LLM_TIMEOUT"] = "60"

# ========================================
# 1. 코드 분석 도구 정의
# ========================================

class CodeAnalysisTool(Tool):
    """코드 정적 분석 도구"""

    def __init__(self):
        super().__init__(
            name="code_analysis",
            description="Python 코드 구조, 복잡도 및 잠재적 문제 분석"
        )

    def run(self, parameters: Dict[str, Any]) -> str:
        """코드를 분석하고 결과 반환"""
        code = parameters.get("code", "")
        if not code:
            return "오류: 코드는 비어 있을 수 없습니다"

        try:
            tree = ast.parse(code)
            functions = [node for node in ast.walk(tree)
                        if isinstance(node, ast.FunctionDef)]
            classes = [node for node in ast.walk(tree)
                      if isinstance(node, ast.ClassDef)]

            result = {
                "함수 수": len(functions),
                "클래스 수": len(classes),
                "코드 줄 수": len(code.split('\n')),
                "함수 목록": [f.name for f in functions],
                "클래스 목록": [c.name for c in classes]
            }
            return str(result)
        except SyntaxError as e:
            return f"구문 오류: {str(e)}"

    def get_parameters(self) -> List[ToolParameter]:
        return [
            ToolParameter(
                name="code",
                type="string",
                description="분석할 Python 코드",
                required=True
            )
        ]

class StyleCheckTool(Tool):
    """코드 스타일 검사 도구"""

    def __init__(self):
        super().__init__(
            name="style_check",
            description="코드가 PEP 8 표준을 준수하는지 검사"
        )

    def run(self, parameters: Dict[str, Any]) -> str:
        """코드 스타일 검사"""
        code = parameters.get("code", "")
        if not code:
            return "오류: 코드는 비어 있을 수 없습니다"

        issues = []
        lines = code.split('\n')
        for i, line in enumerate(lines, 1):
            if len(line) > 79:
                issues.append(f"{i}번 줄: 79자를 초과합니다")
            if line.startswith(' ') and not line.startswith('    '):
                if len(line) - len(line.lstrip()) not in [0, 4, 8, 12]:
                    issues.append(f"{i}번 줄: 비표준 들여쓰기")

        if not issues:
            return "코드 스타일이 양호하며 PEP 8 표준을 준수합니다"
        return "다음 문제가 발견되었습니다:\n" + "\n".join(issues)

    def get_parameters(self) -> List[ToolParameter]:
        return [
            ToolParameter(
                name="code",
                type="string",
                description="검사할 Python 코드",
                required=True
            )
        ]

# ========================================
# 2. 도구 레지스트리 및 에이전트 생성
# ========================================

# 도구 레지스트리 생성
tool_registry = ToolRegistry()
tool_registry.register_tool(CodeAnalysisTool())
tool_registry.register_tool(StyleCheckTool())

# LLM 초기화
llm = HelloAgentsLLM()

# 시스템 프롬프트 정의
system_prompt = """당신은 경험 많은 코드 리뷰 전문가입니다. 당신의 역할은:

1. code_analysis 도구를 사용하여 코드 구조 분석
2. style_check 도구를 사용하여 코드 스타일 검사
3. 분석 결과를 기반으로 상세한 리뷰 보고서 제공

리뷰 보고서에 포함해야 할 사항:
- 코드 구조 분석
- 스타일 문제
- 잠재적 버그
- 성능 최적화 제안
- 모범 사례 권장 사항

보고서를 Markdown 형식으로 출력하세요."""

# 에이전트 생성
agent = SimpleAgent(
    name="코드 리뷰 보조자",
    llm=llm,
    system_prompt=system_prompt,
    tool_registry=tool_registry
)

# ========================================
# 3. 예시 실행
# ========================================

# 샘플 코드 읽기
with open("data/sample_code.py", "r", encoding="utf-8") as f:
    sample_code = f.read()

print("=== 리뷰할 코드 ===")
print(sample_code)
print("\n" + "="*50 + "\n")

# 코드 리뷰 실행
print("=== 코드 리뷰 시작 ===")
review_result = agent.run(f"다음 Python 코드를 리뷰해주세요:\n\n```python\n{sample_code}\n```")

print(review_result)

# 리뷰 보고서 저장
with open("outputs/review_report.md", "w", encoding="utf-8") as f:
    f.write(review_result)

print("\n리뷰 보고서가 outputs/review_report.md에 저장되었습니다")
```

**README.md 예시**

```markdown
# CodeReviewAgent - 지능형 코드 리뷰 보조자

> HelloAgents 프레임워크 기반의 지능형 코드 리뷰 도구

## 📝 프로젝트 소개

CodeReviewAgent는 Python 코드 품질을 자동으로 분석하고, 잠재적 문제를 발견하며, 최적화 제안을 제공하는 지능형 코드 리뷰 보조자입니다.

### 핵심 기능

- ✅ 코드 구조 분석: 함수, 클래스, 코드 줄 수 등을 집계
- ✅ 스타일 검사: PEP 8 표준 준수 여부 확인
- ✅ 지능형 제안: LLM을 기반으로 심층 분석 및 최적화 제안 제공
- ✅ 보고서 생성: Markdown 형식의 리뷰 보고서 생성

## 🛠️ 기술 스택

- HelloAgents 프레임워크 (SimpleAgent + ToolRegistry)
- Python AST 모듈 (코드 파싱)
- ModelScope API (Qwen2.5-72B 모델)

## 🚀 빠른 시작

### 의존성 설치

\`\`\`bash
pip install -r requirements.txt
\`\`\`

### LLM 파라미터 구성

**방법 1: .env 파일 사용**

\`\`\`bash
cp .env.example .env
# .env 파일을 편집하여 API 키 입력
\`\`\`

**방법 2: Notebook에서 직접 설정**

프로젝트는 ModelScope API로 미리 구성되어 있어 바로 실행할 수 있습니다. 수정하려면 main.ipynb의 1부 구성 코드를 편집하세요.

### 프로젝트 실행

\`\`\`bash
jupyter lab
# main.ipynb를 열고 모든 셀 실행
\`\`\`

## 📖 사용 예시

1. 리뷰할 코드를 `data/sample_code.py`에 넣기
2. `main.ipynb` 실행
3. 생성된 리뷰 보고서 `outputs/review_report.md` 확인

## 🎯 프로젝트 하이라이트

- **자동화**: 줄별로 수동 검사할 필요 없이 자동으로 문제 발견
- **지능화**: LLM을 사용하여 코드 의미를 이해하고 심층 제안 제공
- **확장성**: 새로운 검사 규칙과 도구를 쉽게 추가 가능

## 👤 작성자

- GitHub: [@jjyaoao](https://github.com/jjyaoao)
- 프로젝트 링크: [CodeReviewAgent](https://github.com/datawhalechina/hello-agents/tree/main/Co-creation-projects/jjyaoao-CodeReviewAgent)

## 🙏 감사의 말

Datawhale 커뮤니티와 Hello-Agents 프로젝트에 감사드립니다!
```

## 16.7 요약 및 전망

졸업 프로젝트를 완성함으로써 여러분은 에이전트 시스템 설계의 전체 프로세스를 마스터했을 것입니다: 요구사항에서 시스템 아키텍처를 설계하고, HelloAgents 프레임워크의 다양한 기능과 컴포넌트를 능숙하게 사용하고, 에이전트 기능을 확장하기 위한 커스텀 도구를 개발하고, 요구사항 분석부터 코드 구현까지 완전한 프로젝트 개발을 완료하고, 오픈소스 협업을 위한 Git 및 GitHub 사용 방법을 배우고, 명확한 기술 문서를 작성하는 것까지 모두 포함됩니다.

이 프로젝트에서 우리는 HelloAgents 프레임워크를 처음부터 구축하고 이를 사용하여 여러 실용적인 애플리케이션을 구현했습니다. 졸업 프로젝트 완성은 시작에 불과합니다. 더 많은 에이전트 패러다임과 알고리즘, 프롬프트 엔지니어링과 컨텍스트 엔지니어링, 멀티 에이전트 협업 메커니즘 및 기타 이론적 지식을 계속 심화 학습할 수 있습니다. 웹 개발을 배워 완전한 애플리케이션을 구축하고, 데이터베이스를 배워 데이터 지속성을 구현하고, 배포를 배워 애플리케이션을 온라인에 출시하는 방식으로 기술 스택을 확장할 수도 있습니다. 더 많은 기능을 추가하고, 성능과 사용자 경험을 최적화하고, 테스트와 문서를 개선하여 프로젝트를 지속적으로 발전시킬 수 있습니다. 무엇보다도, 다른 학습자를 돕고, Hello-Agents 프레임워크 개발에 참여하고, 경험과 통찰을 공유하여 커뮤니티 기여에 적극적으로 참여하세요.

1장의 단순한 에이전트에서 이제 완전한 멀티 에이전트 애플리케이션을 독립적으로 구축할 수 있게 된 여러분은 흥미진진한 학습 여정을 걸어왔습니다. 하지만 이것은 끝이 아닙니다 - 새로운 시작입니다.

AI 기술은 빠르게 변화하고 있으며, 에이전트 분야는 무한한 가능성으로 가득합니다. 호기심을 유지하고 새로운 기술을 지속적으로 배우고, AI 기술을 용감하게 사용하여 실제 문제를 해결하고 가치를 창출하고, 기꺼이 경험과 성과를 커뮤니티와 공유하고, 탁월함을 추구하며 작품을 지속적으로 다듬어 나가시기를 바랍니다.

마지막으로, 이 프로젝트를 처음부터 끝까지 읽어주셔서 감사합니다. 학습 과정에서 무언가를 얻으셨기를 바라며, 배운 것을 실제 프로젝트에 적용하여 놀라운 에이전트 애플리케이션을 만들어 가시길 바랍니다. AI의 미래는 무한한 가능성으로 가득합니다 - 함께 탐험하고 창조해 나갑시다!

**기억하세요: 배우는 가장 좋은 방법은 직접 실습하는 것입니다!**

이제 나만의 에이전트 애플리케이션을 만들기 시작하세요! Co-creation-projects 디렉토리에서 여러분의 훌륭한 작품을 기대합니다!

Hello-Agents 프로젝트가 도움이 되셨다면 ⭐Star를 눌러주세요!

---
<div align="center">
  <strong>🎓 Hello-Agents 튜토리얼을 완료하신 것을 축하합니다! 🎉</strong>
</div>

