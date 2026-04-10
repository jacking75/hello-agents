# 제13장 지능형 여행 어시스턴트

## 들어가며

앞선 장들에서 우리는 HelloAgents 프레임워크를 처음부터 구축하며, 다양한 지능형 에이전트 패러다임, 도구 시스템, 메모리 메커니즘, 프로토콜 통신, 성능 평가 등 핵심 기능들을 구현했다. 이 장부터는 완전히 새로운 단계에 진입한다: **배운 지식을 하나로 융합하여 완전한 실용 애플리케이션을 구축하는 것이다.**

1장에서 처음 만들었던 에이전트가 기억나는가? 그것은 `Thought-Action-Observation` 루프의 기본 원리를 보여주는 간단한 지능형 여행 어시스턴트였다. 이 장의 지능형 여행 어시스턴트는 완전한 프로젝트로, 다음과 같은 핵심 기능들을 포함한다:

**(1) 지능형 일정 계획**: 사용자가 목적지, 날짜, 선호도 등의 정보를 입력하면 시스템이 명소, 식사, 호텔이 포함된 완전한 여행 일정을 자동으로 생성한다.

**(2) 지도 시각화**: 지도 위에 명소 위치를 표시하고 관람 경로를 그려, 일정을 한눈에 파악할 수 있게 한다.

**(3) 예산 계산**: 입장료, 호텔, 식사, 교통비를 자동으로 계산하여 예산 내역을 표시한다.

**(4) 일정 편집**: 명소 추가, 삭제, 조정을 지원하며, 지도를 실시간으로 업데이트한다.

**(5) 내보내기 기능**: PDF 또는 이미지로 내보내기를 지원하여 저장 및 공유가 편리하다.

---

## **13.1 프로젝트 개요 및 아키텍처 설계**

### **13.1.1 왜 지능형 여행 어시스턴트가 필요한가**

여행 계획을 세우는 일은 설레면서도 머리 아픈 작업이다. 인터넷에서 명소 정보를 검색하고, 여러 여행 가이드를 비교하고, 날씨 예보를 확인하고, 호텔을 예약하고, 예산을 계산하고, 경로를 계획해야 한다. 이 과정은 몇 시간에서 심지어 며칠이 걸릴 수 있다. 그리고 그 많은 시간을 투자하고 나서도, 계획한 일정이 합리적인지, 중요한 명소를 빠뜨리지 않았는지, 예산이 정확한지 확신하기 어렵다.

전통적인 여행 계획 방식에는 몇 가지 고질적인 문제점이 있다. 첫 번째는 **정보 분산** 문제다. 명소 정보는 여행 사이트에, 날씨 정보는 날씨 사이트에, 호텔 정보는 예약 사이트에 각각 흩어져 있어, 여러 사이트를 넘나들며 이 정보들을 수동으로 통합해야 한다. 두 번째는 **개인화 부족** 문제다. 대부분의 여행 가이드는 범용적으로 작성되어 있어, 개인의 선호도, 예산 제약, 여행 기간 등을 고려하지 않는다. 마지막으로 **조정의 어려움**이 있다. 일정을 수정하고 싶을 때, 명소 순서, 시간 배분, 예산이 모두 서로 연결되어 있기 때문에 전체 일정을 다시 계획해야 할 수도 있다.

AI 기술은 이러한 문제를 해결할 새로운 가능성을 제시한다. "베이징에서 3일 동안 역사와 문화를 즐기고 싶고, 예산은 중간 정도야"라고 시스템에게 말하는 것만으로, 시스템이 자동으로 완전한 여행 계획을 생성해준다고 상상해보라. 매일 어떤 명소를 방문하고, 어디서 밥을 먹고, 어떤 호텔에 묵을지, 예산은 얼마가 필요한지를 모두 포함해서 말이다. 게다가 이 계획은 수정도 가능하다. 마음에 들지 않는 명소를 삭제하거나 관람 순서를 조정하면, 시스템이 자동으로 지도와 예산을 업데이트한다.

이것이 우리가 구축할 지능형 여행 어시스턴트다. 단순한 기술 시연이 아니라, 진정으로 유용한 애플리케이션이다. 이 프로젝트를 통해 AI 기술을 실제 문제에 어떻게 적용하는지, 다중 에이전트 시스템을 어떻게 설계하는지, 완전한 웹 애플리케이션을 어떻게 구축하는지를 배울 수 있다.

### **13.1.2 기술 아키텍처 개요**

시스템은 고전적인 **프론트엔드-백엔드 분리 아키텍처**를 채택하며, 그림 13.1과 같이 네 개의 계층으로 나뉜다:

**(1) 프론트엔드 계층 (Vue3+TypeScript)**: 사용자 상호작용과 데이터 표시를 담당하며, 폼 입력, 결과 표시, 지도 시각화를 포함한다.

**(2) 백엔드 계층 (FastAPI)**: API 라우팅, 데이터 검증, 비즈니스 로직을 담당한다.

**(3) 에이전트 계층 (HelloAgents)**: 작업 분해, 도구 호출, 결과 통합을 담당한다. 4개의 전문 에이전트를 포함한다.

**(4) 외부 서비스 계층**: 데이터와 기능을 제공하며, 고덕 지도 API, Unsplash API, LLM API를 포함한다.

데이터 흐름은 다음과 같다: 사용자가 프론트엔드에서 폼 작성 → 백엔드에서 데이터 검증 → 에이전트 시스템 호출 → 에이전트가 순서대로 명소 검색, 날씨 조회, 호텔 추천, 일정 계획 에이전트 호출 → 각 에이전트가 MCP 프로토콜을 통해 외부 API 호출 → 결과를 통합하여 프론트엔드에 반환 → 프론트엔드에서 렌더링 및 표시.

프로젝트 구조는 다음과 같으며, 소스 코드 위치 파악에 편리하다:

```
helloagents-trip-planner/
├── backend/                    # 백엔드 코드
│   ├── app/
│   │   ├── agents/            # 에이전트 구현
│   │   ├── api/               # API 라우팅
│   │   ├── models/            # 데이터 모델
│   │   ├── services/          # 서비스 계층
│   │   └── config.py          # 설정 파일
│   └── requirements.txt       # Python 의존성
│
└── frontend/                   # 프론트엔드 코드
    ├── src/
    │   ├── views/             # 페이지 컴포넌트
    │   ├── services/          # API 서비스
    │   ├── types/             # 타입 정의
    │   └── router/            # 라우터 설정
    └── package.json           # npm 의존성
```

상세한 아키텍처 설계와 데이터 흐름은 이후 절에서 소개한다.

### **13.1.3 빠른 체험: 5분 만에 프로젝트 실행하기**

구현 세부 사항을 깊이 학습하기 전에, 먼저 프로젝트를 실행해보고 최종 결과를 확인해보자. 이렇게 하면 전체 시스템에 대한 직관적인 이해를 얻을 수 있다.

**환경 요구사항:**
- Python 3.10 이상
- Node.js 16.0 이상
- npm 8.0 이상

**API 키 준비:**

다음 API 키들이 필요하다:
- LLM API (OpenAI, DeepSeek 등)
- 고덕 지도 Web 서비스 키: https://console.amap.com/ 에서 등록 후 애플리케이션 생성
- Unsplash Access Key: https://unsplash.com/developers 에서 등록 후 애플리케이션 생성

모든 API 키를 `.env` 파일에 넣는다.

백엔드 시작:

```bash
# 1. 백엔드 디렉토리로 이동
cd helloagents-trip-planner/backend

# 2. 의존성 설치
pip install -r requirements.txt

# 3. 환경 변수 설정
cp .env.example .env
# .env 파일을 편집하여 API 키 입력

# 4. 백엔드 서비스 시작
uvicorn app.api.main:app --reload
# 또는
python run.py
```

성공적으로 시작되면, http://localhost:8000/docs 에서 API 문서를 확인할 수 있다.

새 터미널 창을 열어:

```bash
# 1. 프론트엔드 디렉토리로 이동
cd helloagents-trip-planner/frontend

# 2. 의존성 설치
npm install

# 3. 프론트엔드 서비스 시작
npm run dev
```

성공적으로 시작되면, http://localhost:5173 에서 애플리케이션을 사용할 수 있다.

핵심 기능 체험: 먼저 홈페이지 폼에서 목적지 도시, 여행 날짜, 선호도, 예산, 교통 및 숙박 유형 등의 정보를 입력한다. "계획 시작" 버튼을 클릭하면 시스템이 로딩 진행 표시줄을 보여주고, 곧 결과 페이지를 생성한다(그림 13.2 참조).

이후 로딩이 완료되면, 해당 페이지에서 일정 개요, 예산 내역, 명소 지도, 일별 일정 세부 내용, 날씨 정보가 명확하게 표시된다(그림 13.3, 13.4 참조).

사용자가 개인화된 조정이 필요한 경우, "일정 편집" 버튼을 클릭하여 명소 순서를 자유롭게 조정하거나 특정 명소를 삭제할 수 있다(그림 13.5 참조). 계획이 완료되면, "일정 내보내기" 드롭다운 메뉴를 통해 최종 계획을 이미지 또는 PDF 파일로 손쉽게 저장하여 언제든지 확인할 수 있다.

---

## **13.2 데이터 모델 설계**

### **13.2.1 웹 애플리케이션에서의 데이터 흐름**

지능형 여행 어시스턴트를 구축할 때, 우리는 핵심 문제를 해결해야 한다: **여행 계획 데이터를 어떻게 표현하고 전달할 것인가?**

완전한 웹 애플리케이션에서 데이터가 어떻게 흐르는지 이해해야 한다. 사용자가 브라우저에서 "계획 시작" 버튼을 클릭할 때 무슨 일이 벌어지는지 상상해보자.

사용자가 프론트엔드에서 입력한 폼 데이터(목적지, 날짜, 예산 등)는 HTTP 요청을 통해 백엔드 서버로 전송되어야 한다. 백엔드가 데이터를 수신하면 에이전트 시스템을 호출하여 처리한다. 에이전트는 다시 고덕 지도 API, Unsplash API 등 외부 서비스를 호출하여 데이터를 가져온다. 이러한 외부 API가 반환하는 데이터 형식은 제각각으로, 어떤 것은 `lng`, 어떤 것은 `lon`, 또 어떤 것은 `longitude`를 사용한다. 마지막으로, 백엔드는 처리된 데이터를 프론트엔드에 반환하고, 프론트엔드는 이를 사용자가 보는 페이지로 렌더링한다.

이 과정에서 데이터는 여러 번의 변환을 거친다: 프론트엔드 폼 → HTTP 요청 → 백엔드 Python 객체 → 외부 API 응답 → 백엔드 Python 객체 → HTTP 응답 → 프론트엔드 TypeScript 객체 → 페이지 표시. 통일된 데이터 형식이 없다면, 각 변환 단계마다 오류가 발생할 수 있다. 이것이 **데이터 모델**이 필요한 이유다.

### **13.2.2 딕셔너리에서 Pydantic 모델로**

1장의 간단한 프로토타입부터 시작해보자. 그 프로토타입에서 우리는 Python 딕셔너리를 사용하여 명소 데이터를 표현했다:

```python
# 1장의 방식: 딕셔너리 사용
attraction = {
    "name": "고궁",
    "location": {"lng": 116.397128, "lat": 39.916527},
    "price": 60
}

# 데이터 접근
lng = attraction["location"]["lng"]
```

이 방식은 프로토타입 단계에서는 매우 편리하지만, 실제 프로젝트에서는 많은 문제를 만나게 된다. 첫 번째는 **필드명 불통일** 문제다. 고덕 지도 API가 반환하는 위치 데이터는 `"116.397128,39.916527"` 같은 문자열로, 위경도로 수동으로 분리해야 한다. 반면 Unsplash API는 `longitude`와 `latitude`를 사용할 수도 있다. 코드 여기저기서 딕셔너리를 사용한다면, 매 곳마다 이러한 차이를 처리해야 한다.

두 번째는 **타입 안전성** 문제다. 실수로 `price`를 문자열 `"60"`으로 입력했다고 가정해보자. Python에서는 즉시 오류가 발생하지 않지만, 총 예산을 계산할 때 문제가 생긴다. 더 나쁜 것은, 이런 오류는 런타임에만 발견할 수 있고, 오류 메시지도 위치를 파악하기 어려울 수 있다는 점이다.

마지막으로 **유지보수성** 문제다. 명소에 새로운 필드(예: `rating` 평점)를 추가해야 할 때, 코드의 여러 곳을 수정해야 한다. 어느 한 곳을 놓치면 데이터 불일치가 발생한다.

Pydantic이 해결책을 제공한다. 이것은 Python의 데이터 검증 라이브러리로, 클래스를 사용하여 데이터 구조를 정의하고, 검증, 변환, 직렬화를 자동으로 처리할 수 있게 해준다. 간단한 예시를 살펴보자:

```python
from pydantic import BaseModel, Field

class Location(BaseModel):
    longitude: float = Field(..., description="경도")
    latitude: float = Field(..., description="위도")

class Attraction(BaseModel):
    name: str
    location: Location
    ticket_price: int = 0

# 객체 생성
attraction = Attraction(
    name="고궁",
    location=Location(longitude=116.397128, latitude=39.916527),
    ticket_price=60
)

# 타입 안전한 접근
lng = attraction.location.longitude  # IDE가 코드 자동 완성 제공
```

이렇게 하면 몇 가지 장점이 있다. 첫째, 잘못된 타입(예: `ticket_price`를 문자열로 설정)을 전달하면 Pydantic이 즉시 예외를 던지며 어디서 오류가 발생했는지 알려준다. 둘째, IDE가 타입 정의를 기반으로 코드 자동 완성과 타입 검사를 제공하여 오타를 크게 줄여준다. 마지막으로, 데이터 구조를 수정해야 할 때 클래스 정의만 수정하면, 이 클래스를 사용하는 모든 곳이 자동으로 업데이트된다.

### **13.2.3 Pydantic의 핵심 개념**

데이터 모델 설계에 본격적으로 들어가기 전에, Pydantic의 몇 가지 핵심 개념을 먼저 알아보자. Pydantic의 기반은 `BaseModel` 클래스이며, 모든 데이터 모델은 이 클래스를 상속해야 한다. 각 필드에 타입을 지정할 수 있으며, Pydantic이 자동으로 타입 검사와 변환을 수행한다.

필드 정의는 `Field` 함수를 사용하며, 기본값, 설명, 검증 규칙 등을 지정할 수 있다. `...`는 이 필드가 필수임을 의미하며, 객체 생성 시 이 필드를 제공하지 않으면 Pydantic이 예외를 던진다. `Optional`을 사용하여 선택적 필드를 나타내거나, 직접 기본값을 제공할 수도 있다.

```python
from pydantic import BaseModel, Field
from typing import Optional, List

class Attraction(BaseModel):
    name: str = Field(..., description="명소 이름")  # 필수
    rating: float = Field(default=0.0, ge=0, le=5)  # 기본값, 범위 검증
    visit_duration: int = Field(default=60, gt=0)  # 0보다 커야 함
    description: Optional[str] = None  # 선택적 필드
```

Pydantic은 중첩 모델과 리스트도 지원한다. 한 모델 안에서 다른 모델을 필드 타입으로 사용할 수 있어, 복잡한 데이터 구조를 구축할 수 있다. 예를 들어, 명소에는 위치 정보가 포함되고, 일정에는 여러 명소가 포함된다.

```python
class DayPlan(BaseModel):
    date: str
    attractions: List[Attraction]  # 명소 목록
    hotel: Optional[Hotel] = None  # 선택적 호텔 정보
```

가장 강력한 기능 중 하나는 **커스텀 검증기**다. 외부 API가 반환하는 데이터 형식이 요구사항에 맞지 않을 때, `field_validator` 데코레이터를 사용하여 커스텀 검증 및 변환 로직을 정의할 수 있다. 예를 들어, 고덕 지도가 반환하는 온도는 `"16°C"` 같은 문자열인데, 이를 숫자로 변환해야 한다:

```python
from pydantic import field_validator

class WeatherInfo(BaseModel):
    temperature: int
    
    @field_validator('temperature', mode='before')
    def parse_temperature(cls, v):
        """온도 문자열 파싱: "16°C" -> 16"""
        if isinstance(v, str):
            v = v.replace('°C', '').replace('℃', '').strip()
            return int(v)
        return v
```

이 검증기는 객체 생성 전에 자동으로 실행되어 문자열을 정수로 변환한다. 이렇게 하면 코드의 모든 곳에서 온도 형식을 수동으로 처리할 필요가 없다.

### **13.2.4 하향식 모델 설계**

이제 지능형 여행 어시스턴트의 데이터 모델 설계를 시작해보자. 좋은 설계 원칙은 **하향식(bottom-up)**: 먼저 가장 기초적인 모델을 정의하고, 점차 이를 조합하여 복잡한 구조를 만드는 것이다. 이렇게 하면 각 모델이 단순하여 이해와 유지보수가 쉽다.

가장 기초적인 모델은 **위치 정보**다. 명소, 호텔, 식당 모두 위치 정보가 필요하다. 경위도 좌표를 나타내는 `Location` 클래스를 정의한다:

```python
class Location(BaseModel):
    """위치 정보(경위도 좌표)"""
    longitude: float = Field(..., description="경도", ge=-180, le=180)
    latitude: float = Field(..., description="위도", ge=-90, le=90)
```

여기서 범위 검증(`ge`는 이상, `le`는 이하)을 사용하여 경위도 값이 합리적인 범위 내에 있도록 보장한다.

다음은 **명소 정보**다. 명소에는 이름, 주소, 위치, 관람 시간, 설명, 평점, 이미지, 입장료 등이 포함된다. `Location`을 필드 타입으로 사용하는 중첩 모델이다:

```python
class Attraction(BaseModel):
    """명소 정보"""
    name: str = Field(..., description="명소 이름")
    address: str = Field(..., description="주소")
    location: Location = Field(..., description="경위도 좌표")
    visit_duration: int = Field(..., description="권장 관람 시간(분)", gt=0)
    description: str = Field(..., description="명소 설명")
    category: Optional[str] = Field(default="명소", description="명소 카테고리")
    rating: Optional[float] = Field(default=None, ge=0, le=5, description="평점")
    image_url: Optional[str] = Field(default=None, description="이미지 URL")
    ticket_price: int = Field(default=0, ge=0, description="입장료(위안)")
```

유사하게, **식사 정보**와 **호텔 정보**를 정의한다. 이 모델들의 구조는 매우 비슷하며, 이름, 주소, 위치, 비용 등의 기본 정보를 포함한다:

```python
class Meal(BaseModel):
    """식사 정보"""
    type: str = Field(..., description="식사 유형: breakfast/lunch/dinner/snack")
    name: str = Field(..., description="식당 이름")
    address: Optional[str] = Field(default=None, description="주소")
    location: Optional[Location] = Field(default=None, description="경위도 좌표")
    description: Optional[str] = Field(default=None, description="설명")
    estimated_cost: int = Field(default=0, description="예상 비용(위안)")

class Hotel(BaseModel):
    """호텔 정보"""
    name: str = Field(..., description="호텔 이름")
    address: str = Field(default="", description="호텔 주소")
    location: Optional[Location] = Field(default=None, description="호텔 위치")
    price_range: str = Field(default="", description="가격 범위")
    rating: str = Field(default="", description="평점")
    distance: str = Field(default="", description="명소까지의 거리")
    type: str = Field(default="", description="호텔 유형")
    estimated_cost: int = Field(default=0, description="예상 비용(위안/박)")
```

**예산 정보**는 특별한 모델로, 위치 정보 대신 각 항목 비용의 합계를 포함한다:

```python
class Budget(BaseModel):
    """예산 정보"""
    total_attractions: int = Field(default=0, description="명소 입장료 총비용")
    total_hotels: int = Field(default=0, description="호텔 총비용")
    total_meals: int = Field(default=0, description="식사 총비용")
    total_transportation: int = Field(default=0, description="교통 총비용")
    total: int = Field(default=0, description="총비용")
```

이제 이 기초 모델들을 조합하여 **일별 일정**을 구성할 수 있다. 일별 일정에는 날짜, 설명, 교통 수단, 숙박 안내, 호텔, 명소 목록, 식사 목록이 포함된다:

```python
class DayPlan(BaseModel):
    """일별 일정"""
    date: str = Field(..., description="날짜")
    day_index: int = Field(..., description="몇 번째 날(0부터 시작)")
    description: str = Field(..., description="당일 일정 설명")
    transportation: str = Field(..., description="교통 수단")
    accommodation: str = Field(..., description="숙박 안내")
    hotel: Optional[Hotel] = Field(default=None, description="호텔 정보")
    attractions: List[Attraction] = Field(default_factory=list, description="명소 목록")
    meals: List[Meal] = Field(default_factory=list, description="식사 안내")
```

`List[Attraction]`을 사용하여 명소 목록을 나타내고, `default_factory=list`는 기본값이 빈 리스트임을 의미한다.

**날씨 정보**는 고덕 지도가 반환하는 온도 형식이 규격화되어 있지 않아 특별한 처리가 필요하다. 커스텀 검증기를 사용하여 처리한다:

```python
class WeatherInfo(BaseModel):
    """날씨 정보"""
    date: str = Field(..., description="날짜")
    day_weather: str = Field(..., description="낮 날씨")
    night_weather: str = Field(..., description="밤 날씨")
    day_temp: int = Field(..., description="낮 기온(섭씨)")
    night_temp: int = Field(..., description="밤 기온(섭씨)")
    wind_direction: str = Field(..., description="풍향")
    wind_power: str = Field(..., description="풍속")
    
    @field_validator('day_temp', 'night_temp', mode='before')
    def parse_temperature(cls, v):
        """온도 문자열 파싱: "16°C" -> 16"""
        if isinstance(v, str):
            v = v.replace('°C', '').replace('℃', '').replace('°', '').strip()
            try:
                return int(v)
            except ValueError:
                return 0  # 오류 처리
        return v
```

마지막으로, **완전한 여행 계획**을 정의한다. 이것이 최상위 모델로, 모든 정보를 포함한다:

```python
class TripPlan(BaseModel):
    """여행 계획"""
    city: str = Field(..., description="목적지 도시")
    start_date: str = Field(..., description="시작 날짜")
    end_date: str = Field(..., description="종료 날짜")
    days: List[DayPlan] = Field(default_factory=list, description="일별 일정")
    weather_info: List[WeatherInfo] = Field(default_factory=list, description="날씨 정보")
    overall_suggestions: str = Field(..., description="전체적인 제안")
    budget: Optional[Budget] = Field(default=None, description="예산 정보")
```

이렇게 전체 데이터 모델 설계가 완성된다. 가장 기초적인 `Location`에서 `Attraction`, `Meal`, `Hotel`을 거쳐 `DayPlan`으로, 그리고 최종적으로 `TripPlan`까지, 명확한 계층 구조를 형성한다.

### **13.2.5 데이터 모델의 웹 애플리케이션 적용**

이제 이 데이터 모델들이 실제 웹 애플리케이션에서 어떻게 사용되는지 살펴보자. FastAPI에서 Pydantic 모델은 요청과 응답의 타입 정의로 직접 사용될 수 있다. FastAPI는 데이터 검증, 직렬화, 문서 생성을 자동으로 처리한다.

```python
from fastapi import FastAPI
from app.models.schemas import TripPlanRequest, TripPlan

app = FastAPI()

@app.post("/api/trip/plan", response_model=TripPlan)
async def create_trip_plan(request: TripPlanRequest) -> TripPlan:
    """
    여행 계획 생성
    
    FastAPI 자동 처리:
    1. 요청 데이터 검증(TripPlanRequest)
    2. 응답 데이터 검증(TripPlan)
    3. OpenAPI 문서 생성
    """
    trip_plan = await generate_trip_plan(request)
    return trip_plan
```

사용자가 `POST /api/trip/plan`에 요청을 보내면, FastAPI가 자동으로 JSON 데이터를 `TripPlanRequest` 객체로 변환한다. 데이터 형식이 올바르지 않으면(예: 필수 필드 누락, 타입 불일치) FastAPI가 자동으로 400 오류를 반환하고 어디가 잘못됐는지 알려준다.

프론트엔드에서도 대응하는 TypeScript 타입을 정의해야 한다. TypeScript와 Python은 다른 언어지만 데이터 구조는 동일하다:

```typescript
interface Location {
  longitude: number;
  latitude: number;
}

interface Attraction {
  name: string;
  address: string;
  location: Location;
  visit_duration: number;
  ticket_price: number;
}

interface TripPlan {
  city: string;
  start_date: string;
  end_date: string;
  days: DayPlan[];
}
```

이렇게 프론트엔드와 백엔드가 통일된 데이터 형식을 사용한다. 백엔드가 `TripPlan` 객체를 반환할 때, 프론트엔드는 별도의 변환 없이 직접 사용할 수 있다. TypeScript의 타입 검사도 많은 오류를 방지하는 데 도움이 된다.

---

## **13.3 다중 에이전트 협업 설계**

### **13.3.1 왜 다중 에이전트가 필요한가**

7장에서 `SimpleAgent`를 사용하여 에이전트를 구축하는 방법을 배웠다. `SimpleAgent`의 설계 이념은 단순하고 직접적이다: `run()` 메서드를 호출할 때마다 에이전트가 사용자의 질문을 분석하고, 도구를 호출할지 결정한 후 결과를 반환한다. 이 설계는 간단한 작업을 처리할 때 매우 효과적이지만, 여행 계획 같은 작업을 처리할 때는 몇 가지 문제가 발생한다.

단일 에이전트로 여행 계획을 완성한다면, 이 에이전트는 무엇을 해야 할까? 먼저 명소 정보를 검색하기 위해 고덕 지도의 POI 검색 도구를 호출해야 한다. 그다음 날씨 정보를 조회하기 위해 날씨 조회 도구를 호출해야 한다. 그런 다음 호텔 정보를 검색하기 위해 다시 POI 검색 도구를 호출해야 한다. 마지막으로 이 모든 정보를 통합하여 완전한 여행 계획을 생성해야 한다.

단순해 보이지만 실제로는 첫 번째 문제에 부딪힌다: **도구 호출의 제한**이다. `SimpleAgent`는 `run()` 호출 한 번에 하나의 도구만 실행할 수 있다. 즉, `run()` 메서드를 여러 번 호출해야 하고, 매번 하나의 작업을 처리한다. 하지만 이렇게 하면 새로운 문제가 생긴다: 여러 번의 호출 사이에 정보를 어떻게 전달할 것인가? 첫 번째 호출로 얻은 명소 정보를 두 번째 호출에 어떻게 전달할 것인가? 이 중간 결과들을 수동으로 관리해야 하므로 코드가 매우 복잡해진다.

물론 `ReactAgent`를 사용하여 이 문제를 해결할 수 있다. `ReactAgent`는 한 번의 호출에서 여러 도구를 실행할 수 있으며, 자동으로 여러 라운드의 사고와 행동을 수행한다. 하지만 이는 새로운 문제를 야기한다: **시간 비용**이다. `ReactAgent`의 각 라운드 사고는 LLM 호출이 필요하다. 세 개의 도구를 호출해야 한다면 최소 세 라운드의 사고, 즉 최소 세 번의 LLM 호출이 필요하다. 게다가 이 호출들은 직렬로 처리되므로, 전 단계가 완료되어야 다음 단계가 시작될 수 있어 전체 소요 시간이 매우 길어진다.

두 번째 문제는 **프롬프트의 복잡성**이다. 하나의 에이전트가 모든 작업을 완료하려면, 프롬프트에 각 작업의 실행 로직을 상세히 기술해야 한다. 예를 들면:

```python
COMPLEX_PROMPT = """당신은 여행 계획 어시스턴트입니다. 다음을 수행해야 합니다:
1. maps_text_search를 사용하여 명소를 검색하세요. 키워드는 사용자 선호도에 따라 결정
2. maps_weather를 사용하여 날씨를 조회하세요. 향후 며칠의 날씨 예보 획득
3. maps_text_search를 사용하여 호텔을 검색하세요. 유형은 사용자 요구에 따라 결정
4. 모든 정보를 통합하여 여행 계획을 생성하세요. 매일의 명소, 식사, 숙박 안배 포함
주의: 반드시 순서대로 실행하고, 각 도구는 한 번만 호출하며, 출력은 JSON 형식이어야 함...
"""
```

이런 프롬프트에는 몇 가지 문제가 있다. 첫째, **유지보수가 어렵다**. 명소 검색 로직을 수정하려면(예: 평점 필터 추가) 전체 프롬프트를 수정해야 하며, 다른 부분에 영향을 미치기 쉽다. 둘째, **오류가 발생하기 쉽다**. LLM이 여러 작업의 요구사항을 동시에 이해해야 하므로, 서로 다른 작업의 형식과 매개변수를 혼동하기 쉽다. 마지막으로 **디버깅이 어렵다**. 생성된 계획이 기대에 미치지 못할 때, 어느 단계에서 문제가 발생했는지 파악하기 어렵다. 명소 검색이 부정확한 것인지, 날씨 조회가 실패한 것인지, 통합 로직에 문제가 있는 것인지 알 수 없다.

이런 문제들을 직면하면서, 자연스럽게 떠오르는 생각이 있다: 복잡한 작업을 여러 개의 간단한 작업으로 분해하고, 서로 다른 에이전트가 각자의 역할을 맡으면 어떨까? 이것이 다중 에이전트 협업의 핵심 사상이다.

현실 세계의 여행사를 상상해보라. 여행사에 여행 계획을 문의하러 갔을 때, 한 사람이 모든 것을 처리하지 않는다. 보통 명소를 추천하는 전문 명소 상담사, 호텔 예약을 담당하는 호텔 상담사, 모든 정보를 완전한 일정으로 통합하는 일정 계획사가 있다. 각 사람은 자신이 잘하는 분야에 집중하고, 마지막에 일정 계획사가 모든 정보를 취합한다. 이런 분업 협업 방식은 한 사람이 모든 것을 하는 것보다 훨씬 효율적이다.

### **13.3.2 에이전트 역할 설계**

작업 분해 원칙에 기반하여, 그림 13.6과 같이 네 개의 전문 에이전트를 설계했다:

- **AttractionSearchAgent(명소 검색 전문가)**: 명소 정보 검색에 집중한다. 사용자의 선호도(예: "역사 문화", "자연 경관")를 이해하고, 고덕 지도의 POI 검색 도구를 호출하여 관련 명소 목록을 반환한다. 프롬프트는 간단하여, 선호도에 따라 키워드를 어떻게 선택하고 도구를 호출하는지만 설명하면 된다.

- **WeatherQueryAgent(날씨 조회 전문가)**: 날씨 정보 조회에 집중한다. 도시 이름만 알면 날씨 조회 도구를 호출하여 향후 며칠의 날씨 예보를 반환한다. 작업이 매우 명확하여 거의 오류가 발생하지 않는다.

- **HotelAgent(호텔 추천 전문가)**: 호텔 정보 검색에 집중한다. 사용자의 숙박 요구(예: "경제형", "고급형")를 이해하고, POI 검색 도구를 호출하여 요구에 맞는 호텔 목록을 반환한다.

- **PlannerAgent(일정 계획 전문가)**: 모든 정보를 통합하는 역할을 담당한다. 앞의 세 에이전트의 출력과 사용자의 원래 요구(날짜, 예산 등)를 받아 완전한 여행 계획을 생성한다. 외부 도구를 호출할 필요 없이 정보 통합과 일정 배치에만 집중한다.

이제 각 에이전트의 역할과 프롬프트를 상세히 설계해보자. 프롬프트 설계 시 몇 가지 핵심 질문을 고려해야 한다: 이 에이전트는 어떤 입력이 필요한가? 어떤 출력을 생성해야 하는가? 어떤 도구를 호출해야 하는가? 어떤 문제를 만날 수 있는가?

**AttractionSearchAgent**의 작업은 사용자 선호도에 따라 명소를 검색하는 것이다. 입력은 도시 이름과 사용자 선호도(예: "역사 문화", "자연 경관")이다. `amap_maps_text_search` 도구를 호출해야 하며, 매개변수는 키워드와 도시다. 출력은 이름, 주소, 평점 등이 포함된 명소 목록이다.

```python
ATTRACTION_AGENT_PROMPT = """당신은 명소 검색 전문가입니다.

**도구 호출 형식:**
`[TOOL_CALL:amap_maps_text_search:keywords=명소,city=도시명]`

**예시:**
- `[TOOL_CALL:amap_maps_text_search:keywords=명소,city=베이징]`
- `[TOOL_CALL:amap_maps_text_search:keywords=박물관,city=상하이]`

**중요:**
- 반드시 도구를 사용하여 검색하고, 정보를 임의로 만들어내지 마세요
- 사용자 선호도({preferences})에 따라 {city}의 명소를 검색하세요
"""
```

이 프롬프트는 간결하지만 필요한 모든 정보를 포함한다. 도구 호출 형식을 명확히 설명하고, 구체적인 예시를 제공하며, 두 가지 중요 원칙도 강조한다: 반드시 도구를 사용할 것(임의로 만들어내지 말 것), 사용자 선호도에 따라 검색할 것.

**WeatherQueryAgent**의 작업은 더 단순하여, 날씨만 조회하면 된다. 입력은 도시 이름이고, 출력은 날씨 정보다.

```python
WEATHER_AGENT_PROMPT = """당신은 날씨 조회 전문가입니다.

**도구 호출 형식:**
`[TOOL_CALL:amap_maps_weather:city=도시명]`

{city}의 날씨 정보를 조회하세요.
"""
```

**HotelAgent**의 작업은 호텔을 검색하는 것이다. 입력은 도시 이름과 숙박 유형이고, 출력은 호텔 목록이다.

```python
HOTEL_AGENT_PROMPT = """당신은 호텔 추천 전문가입니다.

**도구 호출 형식:**
`[TOOL_CALL:amap_maps_text_search:keywords=호텔,city=도시명]`

{city}의 {accommodation} 호텔을 검색하세요.
"""
```

**PlannerAgent**는 가장 복잡하다. 모든 정보를 통합해야 하기 때문이다. 입력은 사용자 요구와 앞의 세 에이전트의 출력이고, 출력은 완전한 여행 계획(JSON 형식)이다.

```python
PLANNER_AGENT_PROMPT = """당신은 일정 계획 전문가입니다.

**출력 형식:**
다음 JSON 형식을 엄격히 준수하여 반환하세요:
{
  "city": "도시 이름",
  "start_date": "YYYY-MM-DD",
  "end_date": "YYYY-MM-DD",
  "days": [...],
  "weather_info": [...],
  "overall_suggestions": "전체적인 제안",
  "budget": {...}
}

**계획 요구사항:**
1. weather_info는 매일의 날씨를 포함해야 함
2. 온도는 순수 숫자로 표시(°C 제외)
3. 매일 2~3개의 명소 배치
4. 명소 간 거리와 관람 시간 고려
5. 아침·점심·저녁 세 끼 식사 포함
6. 실용적인 제안 제공
7. 예산 정보 포함
"""
```

### **13.3.3 에이전트 협업 흐름**

이제 네 에이전트가 어떻게 협업하여 여행 계획 작업을 완성하는지 살펴보자. 전체 흐름은 다섯 단계로 나눌 수 있다:

```python
class TripPlannerAgent:
    def __init__(self):
        self.attraction_agent = SimpleAgent(name="명소 검색", prompt=ATTRACTION_PROMPT)
        self.weather_agent = SimpleAgent(name="날씨 조회", prompt=WEATHER_PROMPT)
        self.hotel_agent = SimpleAgent(name="호텔 추천", prompt=HOTEL_PROMPT)
        self.planner_agent = SimpleAgent(name="일정 계획", prompt=PLANNER_PROMPT)

    def plan_trip(self, request: TripPlanRequest) -> TripPlan:
        # 단계1: 명소 검색
        attraction_response = self.attraction_agent.run(
            f"{request.city}의 {request.preferences} 명소를 검색해주세요"
        )

        # 단계2: 날씨 조회
        weather_response = self.weather_agent.run(
            f"{request.city}의 날씨를 조회해주세요"
        )

        # 단계3: 호텔 추천
        hotel_response = self.hotel_agent.run(
            f"{request.city}의 {request.accommodation} 호텔을 검색해주세요"
        )

        # 단계4: 계획 통합 생성
        planner_query = self._build_planner_query(
            request, attraction_response, weather_response, hotel_response
        )
        planner_response = self.planner_agent.run(planner_query)

        # 단계5: JSON 파싱
        trip_plan = self._parse_trip_plan(planner_response)
        return trip_plan
```

이 흐름은 네 단계를 순서대로 실행하며, 각 단계의 출력이 다음 단계의 입력이 된다. `TripPlanRequest`와 `TripPlan` Pydantic 모델이 사용되고 있음을 주목하자. 이는 13.2절에서 정의한 것들이다.

### **13.3.4 쿼리 구성**

`PlannerAgent`는 모든 정보를 통합해야 하므로, 이 쿼리에는 모든 필요한 정보가 포함되어야 하고, 명확하게 구성되어 LLM이 정확하게 이해할 수 있어야 한다.

```python
def _build_planner_query(
    self,
    request: TripPlanRequest,
    attraction_response: str,
    weather_response: str,
    hotel_response: str
) -> str:
    """계획 에이전트의 쿼리 구성"""
    return f"""
다음 정보를 바탕으로 {request.city}의 {request.days}일 여행 계획을 생성해주세요:

**사용자 요구:**
- 목적지: {request.city}
- 날짜: {request.start_date} ~ {request.end_date}
- 일수: {request.days}일
- 선호도: {request.preferences}
- 예산: {request.budget}
- 교통 수단: {request.transportation}
- 숙박 유형: {request.accommodation}

**명소 정보:**
{attraction_response}

**날씨 정보:**
{weather_response}

**호텔 정보:**
{hotel_response}

매일의 명소 배치, 식사 추천, 숙박 정보, 예산 내역이 포함된 상세한 여행 계획을 생성해주세요.
"""
```

이런 다중 에이전트 협업 설계를 통해, 복잡한 여행 계획 작업을 네 개의 간단한 부분 작업으로 분해했다. 각 에이전트가 자신이 잘하는 분야에 집중하고, 향후 기능 확장(예: 식당 추천 에이전트, 교통 계획 에이전트 추가)을 위한 좋은 기반도 마련했다.

---

## **13.4 MCP 도구 통합 상세 설명**

### **13.4.1 왜 API를 직접 호출하지 않는가**

13.3절에서 여행 계획 작업을 협업하여 완성하는 네 에이전트를 설계했다. 그 중 `AttractionSearchAgent`, `WeatherQueryAgent`, `HotelAgent`는 모두 고덕 지도의 API를 호출하여 데이터를 가져와야 한다. 자연스럽게 드는 질문이 있다: 왜 에이전트 안에서 고덕 지도의 HTTP API를 직접 호출하지 않는가?

직접 API를 호출한다면 어떤 모습인지 살펴보자. 고덕 지도는 POI 검색 API를 제공하며, HTTP 요청을 구성하고 매개변수를 전달하고 응답을 파싱해야 한다:

```python
import requests

def search_poi(keywords: str, city: str, api_key: str):
    """고덕 지도 POI 검색 API 직접 호출"""
    url = "https://restapi.amap.com/v3/place/text"
    params = {
        "keywords": keywords,
        "city": city,
        "key": api_key,
        "output": "json"
    }
    response = requests.get(url, params=params)
    data = response.json()
    return data
```

이 방식은 단순해 보이지만, 실제 사용에서 몇 가지 문제가 발생한다. 첫째, **에이전트의 자율 호출 불가** 문제다. 우리의 HelloAgents 프레임워크에서 에이전트는 프롬프트 내의 도구 호출 마커(예: `[TOOL_CALL:tool_name:arg1=value1]`)를 인식하여 도구를 호출한다. 코드에서 직접 API를 호출하면 에이전트는 자율적인 의사결정 능력을 잃고, 단순한 함수 호출이 된다.

둘째, **매개변수 전달의 복잡성** 문제다. 고덕 지도 API에는 수많은 매개변수가 있다. 예를 들어 POI 검색에는 `keywords`, `city`, `types`, `offset`, `page` 등 십여 개의 매개변수가 있다. 에이전트가 이 매개변수들을 유연하게 사용할 수 있게 하려면 프롬프트에 각 매개변수의 의미와 형식을 상세히 설명해야 하므로 프롬프트가 매우 복잡해진다.

셋째, **응답 파싱의 어려움** 문제다. 고덕 지도 API는 JSON 형식의 데이터를 반환하며 구조가 상당히 복잡하다. 이 데이터를 파싱하여 필요한 필드를 추출하는 코드를 작성해야 한다. API의 응답 형식이 변경되면 파싱 코드도 수정해야 한다.

마지막으로 **도구 관리의 혼란** 문제다. 고덕 지도는 십여 개의 서로 다른 API(POI 검색, 날씨 조회, 경로 계획 등)를 제공한다. 각 API마다 함수를 하나씩 작성하고 에이전트의 도구 목록에 수동으로 등록한다면 코드가 매우 장황해진다. 새 API를 추가하려면 여러 곳을 수정해야 한다.

### **13.4.2 고덕 지도 MCP 통합**

MCP(Model Context Protocol)는 Anthropic이 제안한 표준화 프로토콜로, LLM과 외부 도구를 연결하는 데 사용된다. 이 절에서는 프로젝트에 고덕 지도 MCP 서버를 통합하는 방법을 소개한다. 우리 프로젝트에서는 Node.js로 구현된 MCP 서버인 `amap-mcp-server`를 사용한다.

고덕 지도 MCP 서버는 여러 종류의 도구를 제공하며, 표 13.1과 같이 주요 카테고리로 분류된다.

MCP 프로토콜을 통해 HelloAgents에 매우 편리하게 통합할 수 있다:

```python
from hello_agents.tools import MCPTool
from app.config import get_settings

settings = get_settings()

# MCP 도구 생성
mcp_tool = MCPTool(
    name="amap_mcp",
    command="npx",
    args=["-y", "@sugarforever/amap-mcp-server"],
    env={"AMAP_API_KEY": settings.amap_api_key},
    auto_expand=True
)
```

이 코드는 무엇을 하는가? 먼저 `command`와 `args`는 MCP 서버를 어떻게 시작할지 지정한다. `npx -y @sugarforever/amap-mcp-server`는 npm 저장소에서 `amap-mcp-server` 패키지를 다운로드하고 실행한다. `env` 매개변수는 환경 변수를 전달하며, 여기서는 고덕 지도의 API 키를 전달한다.

**참고:** 이 문서의 일부 예시에서는 `npx`를 사용하여 MCP를 시작한다. 실제 코드 저장소에서는 `uvx` 방식을 사용한다. `npx`와 `uvx`는 설계 이념이 매우 유사하며, 차이는 생태계에만 있다. `npx`는 JavaScript/Node.js(npm의 패키지)에 해당하고, `uvx`는 Python(PyPI의 패키지)에 해당한다. 두 방식 중 어느 것이 더 낫다고 할 수 없으며, 필요에 따라 선택하면 된다.

`MCPTool` 객체를 생성할 때, 백그라운드에서 MCP 서버 프로세스가 시작되고 표준 입출력(stdin/stdout)을 통해 서버와 통신한다. 이것이 MCP 프로토콜의 특징 중 하나다: HTTP 대신 프로세스 간 통신을 사용하여 더 효율적이고 관리가 쉽다.

가장 핵심적인 것은 `auto_expand=True` 매개변수다. True로 설정하면 `MCPTool`이 MCP 서버가 제공하는 도구를 자동으로 쿼리하고, 각 도구에 대해 독립적인 Tool 객체를 생성한다. 그래서 하나의 `MCPTool`만 생성했지만 에이전트는 16개의 도구를 갖게 된다. 이 과정을 살펴보자:

```python
# 하나의 MCPTool 생성
mcp_tool = MCPTool(..., auto_expand=True)
agent.add_tool(mcp_tool)

# 에이전트는 실제로 16개의 도구를 갖게 된다!
print(list(agent.tools.keys()))
# ['amap_maps_text_search', 'amap_maps_weather', ...]
```

그림 13.8처럼, 사용자가 베이징의 명소를 검색하고 싶다고 가정하면, `AttractionSearchAgent`가 "베이징의 역사 문화 명소를 검색해주세요"라는 쿼리를 받는다. 에이전트는 이 쿼리를 분석하여 `amap_maps_text_search` 도구를 호출하기로 결정하고, 매개변수는 `keywords=명소, city=베이징`이다.

에이전트가 도구 호출 마커를 생성한다: `[TOOL_CALL:amap_maps_text_search:keywords=명소,city=베이징]`. HelloAgents 프레임워크가 이 마커를 파싱하여 도구 이름과 매개변수를 추출하고, 해당하는 Tool 객체를 호출한다.

Tool 객체는 `MCPTool`이 자동으로 생성한 것으로, 호출 요청을 MCP 서버로 전송한다. 구체적으로는 JSON-RPC 형식의 메시지를 구성하여 stdin을 통해 서버 프로세스로 전송한다:

```json
{
  "jsonrpc": "2.0",
  "method": "tools/call",
  "params": {
    "name": "amap_maps_text_search",
    "arguments": {
      "keywords": "명소",
      "city": "베이징"
    }
  }
}
```

MCP 서버가 이 메시지를 받아 매개변수를 파싱하고, 고덕 지도의 HTTP API를 호출한다. HTTP 요청을 구성하고, API 키를 추가하고, 요청을 보내고, 응답을 받는다.

고덕 지도 API가 명소 목록, 주소, 좌표 등의 정보가 담긴 JSON 데이터를 반환한다. MCP 서버가 이 데이터를 파싱하여 핵심 필드를 추출하고, 응답 메시지를 구성하여 stdout을 통해 `MCPTool`로 반환한다:

```json
{
  "jsonrpc": "2.0",
  "result": {
    "content": [
      {
        "type": "text",
        "text": "다음 명소를 찾았습니다:\n1. 고궁박물원 - 주소: 둥청구 징산첸제 4호\n2. 톈탄공원 - 주소: 둥청구 톈탄로\n..."
      }
    ]
  }
}
```

`MCPTool`이 응답을 받아 텍스트 내용을 추출하여 에이전트에게 반환한다. 에이전트는 이 결과를 도구 호출의 출력으로 사용하여 최종 응답을 계속 생성한다.

이 흐름이 복잡해 보이지만, 에이전트 입장에서는 `amap_maps_text_search`라는 명소를 검색할 수 있는 도구가 있다는 것만 알면 된다. 모든 하위 계층의 세부 사항은 MCP 프로토콜과 `MCPTool`에 의해 캡슐화되어 있다.

### **13.4.3 MCP 인스턴스 공유**

다중 에이전트 시스템에서 세 에이전트 모두 고덕 지도의 도구를 사용해야 한다. 그렇다면 각 에이전트가 자체 `MCPTool` 인스턴스를 생성해야 할까, 아니면 같은 인스턴스를 공유해야 할까?

각 에이전트가 하나씩 `MCPTool` 인스턴스를 생성하면, 세 개의 서버 프로세스가 동시에 실행된다. 각 프로세스가 독립적으로 고덕 지도 API를 호출하면 API의 속도 제한을 초과할 수 있다. 또한 여러 프로세스가 더 많은 메모리와 CPU 자원을 차지한다.

더 좋은 방법은 모든 에이전트가 하나의 `MCPTool` 인스턴스를 공유하는 것이다. 이렇게 하면 하나의 MCP 서버 프로세스만 시작하면 되고, 모든 API 호출이 이 프로세스를 통해 처리된다. 자원을 절약할 뿐만 아니라 API 호출 빈도도 더 잘 제어할 수 있다.

코드에서는 `TripPlannerAgent`의 생성자에서 하나의 `MCPTool` 인스턴스를 생성하고, 각 하위 에이전트의 도구 목록에 추가한다:

```python
class TripPlannerAgent:
    def __init__(self):
        settings = get_settings()
        self.llm = HelloAgentsLLM()

        # 공유 MCP 도구 인스턴스 생성(한 번만 생성)
        self.mcp_tool = MCPTool(
            name="amap_mcp",
            command="npx",
            args=["-y", "@sugarforever/amap-mcp-server"],
            env={"AMAP_API_KEY": settings.amap_api_key},
            auto_expand=True
        )

        # 여러 에이전트 생성, 동일한 MCP 도구 공유
        self.attraction_agent = SimpleAgent(
            name="AttractionSearchAgent",
            llm=self.llm,
            system_prompt=ATTRACTION_AGENT_PROMPT
        )
        self.attraction_agent.add_tool(self.mcp_tool)  # 공유

        self.weather_agent = SimpleAgent(
            name="WeatherQueryAgent",
            llm=self.llm,
            system_prompt=WEATHER_AGENT_PROMPT
        )
        self.weather_agent.add_tool(self.mcp_tool)  # 공유

        self.hotel_agent = SimpleAgent(
            name="HotelAgent",
            llm=self.llm,
            system_prompt=HOTEL_AGENT_PROMPT
        )
        self.hotel_agent.add_tool(self.mcp_tool)  # 공유
```

이렇게 하면 세 에이전트 모두 고덕 지도의 16개 도구를 사용할 수 있지만, 하위 계층에서는 하나의 MCP 서버 프로세스만 실행된다. `TripPlannerAgent`의 `plan_trip` 메서드를 호출하면, 세 에이전트가 순서대로 도구를 호출하며 모든 요청이 동일한 MCP 서버를 통해 고덕 지도 API로 전송된다.

### **13.4.4 Unsplash 이미지 API 통합**

고덕 지도 외에도, 명소 이미지를 가져와 여행 계획을 더욱 생동감 있게 만들어야 한다. Unsplash API를 사용하여 명소 이미지를 검색한다. 주의할 점은 Unsplash는 해외 서비스이고, 무료로 사용할 수 있는 몇 안 되는 이미지 API 중 하나이지만, 검색 결과가 충분히 정확하지 않을 수 있다는 것이다. 실제 프로젝트에서는 빙, 바이두, 또는 고덕의 POI 이미지 API를 고려할 수 있으나, 이 서비스들은 보통 유료다.

Unsplash API 통합은 비교적 간단하며, `UnsplashService` 클래스를 생성하여 API 호출을 캡슐화한다:

```python
# backend/app/services/unsplash_service.py
import requests
from typing import Optional, List, Dict
import logging

logger = logging.getLogger(__name__)

class UnsplashService:
    """Unsplash 이미지 서비스"""

    def __init__(self, access_key: str):
        self.access_key = access_key
        self.base_url = "https://api.unsplash.com"

    def search_photos(self, query: str, per_page: int = 10) -> List[Dict]:
        """이미지 검색"""
        try:
            url = f"{self.base_url}/search/photos"
            params = {
                "query": query,
                "per_page": per_page,
                "client_id": self.access_key
            }

            response = requests.get(url, params=params, timeout=10)
            response.raise_for_status()

            data = response.json()
            results = data.get("results", [])

            # 이미지 URL 추출
            photos = []
            for result in results:
                photos.append({
                    "url": result["urls"]["regular"],
                    "description": result.get("description", ""),
                    "photographer": result["user"]["name"]
                })

            return photos

        except Exception as e:
            logger.error(f"이미지 검색 실패: {e}")
            return []

    def get_photo_url(self, query: str) -> Optional[str]:
        """단일 이미지 URL 가져오기"""
        photos = self.search_photos(query, per_page=1)
        return photos[0].get("url") if photos else None
```

이 서비스 클래스는 두 가지 메서드를 제공한다: `search_photos`는 여러 이미지를 검색하고, `get_photo_url`은 단일 이미지의 URL을 가져온다. API 라우팅에서 이 서비스를 사용하여 각 명소의 이미지를 가져온다:

```python
# backend/app/api/routes/trip.py
from app.services.unsplash_service import UnsplashService

unsplash_service = UnsplashService(settings.unsplash_access_key)

@router.post("/plan", response_model=TripPlan)
async def create_trip_plan(request: TripPlanRequest) -> TripPlan:
    # 여행 계획 생성
    trip_plan = trip_planner_agent.plan_trip(request)

    # 각 명소에 이미지 가져오기
    for day in trip_plan.days:
        for attraction in day.attractions:
            if not attraction.image_url:
                image_url = unsplash_service.get_photo_url(
                    f"{attraction.name} {trip_plan.city}"
                )
                attraction.image_url = image_url

    return trip_plan
```

Unsplash를 Tool이나 MCP 도구로 캡슐화하지 않고, API 라우팅에서 직접 호출하고 있음을 주목하자. 이는 이미지 검색이 에이전트의 지능적인 의사결정이 필요 없는 단순한 데이터 보강 단계이기 때문이다. 에이전트가 이미지 필요 여부를 자율적으로 결정하거나 다른 이미지 소스를 선택할 수 있게 하려면, Tool로 캡슐화하는 것을 고려할 수 있다.

---

## **13.5 프론트엔드 개발 상세 설명**

### **13.5.1 프론트엔드-백엔드 분리 웹 아키텍처**

프론트엔드 개발을 시작하기 전에, 현대 웹 애플리케이션의 아키텍처 패턴을 이해해야 한다. 초기 웹 개발에서는 프론트엔드와 백엔드가 혼합되어 있었다. PHP, JSP 같은 기술에서 HTML 템플릿과 비즈니스 로직 코드가 같은 파일에 작성되었다. 이 방식은 소규모 프로젝트에서는 편리하지만, 대규모 프로젝트에서는 프론트엔드와 백엔드 개발자가 자주 조율해야 하고, 코드 재사용이 어렵고, 테스트가 어려운 많은 문제가 발생한다.

현대 웹 애플리케이션은 보편적으로 **프론트엔드-백엔드 분리** 아키텍처를 채택한다. 백엔드는 API 인터페이스를 제공하고 JSON 형식의 데이터를 반환하는 역할만 한다. 프론트엔드는 독립적인 애플리케이션으로, HTTP 요청을 통해 백엔드 API를 호출하고, 데이터를 받아 페이지를 렌더링한다. 이 아키텍처에는 몇 가지 명확한 장점이 있다: 프론트엔드와 백엔드가 독립적으로 개발, 배포, 테스트할 수 있다; 프론트엔드는 웹 애플리케이션, 모바일 앱, 데스크탑 앱이 될 수 있으며 모두 동일한 백엔드 API를 사용한다; 프론트엔드는 현대적인 프레임워크와 도구 체인을 사용하여 더 좋은 사용자 경험을 제공할 수 있다.

우리의 지능형 여행 어시스턴트 프로젝트에서 백엔드는 Python과 FastAPI로 구현되었으며, 핵심 API 인터페이스 `POST /api/trip/plan`을 제공한다. 이 인터페이스는 여행 요구를 받아 여행 계획을 반환한다. 프론트엔드는 Vue 3와 TypeScript로 구현된 단일 페이지 애플리케이션(SPA)이다. 사용자가 브라우저에서 폼을 작성하고, "계획 시작" 버튼을 클릭하면, 프론트엔드가 HTTP 요청을 백엔드로 보내고, 응답을 기다린 후 결과 페이지를 렌더링한다. 전체 과정에서 페이지가 새로 고침되지 않아 사용자 경험이 매우 부드럽다.

프론트엔드 기술 스택 선택은 개발 효율성, 성능, 생태계, 학습 곡선 등 여러 요소를 고려해야 한다. 표 13.2와 같이 이 프로젝트에서는 다음 기술 스택을 선택했다.

프로젝트 디렉토리 구조는 다음과 같다:

```
frontend/
├── src/
│   ├── views/              # 페이지 컴포넌트
│   │   ├── Home.vue        # 홈페이지(폼)
│   │   └── Result.vue      # 결과 페이지
│   ├── services/           # API 서비스
│   │   └── api.ts
│   ├── types/              # 타입 정의
│   │   └── index.ts
│   ├── router/             # 라우터 설정
│   │   └── index.ts
│   ├── App.vue
│   └── main.ts
├── package.json
├── vite.config.ts
└── tsconfig.json
```

`views` 디렉토리에는 페이지 컴포넌트, `services` 디렉토리에는 API 호출 로직, `types` 디렉토리에는 TypeScript 타입 정의, `router` 디렉토리에는 라우팅 설정이 들어있다.

### **13.5.2 타입 정의**

13.2절에서 백엔드에서 `Location`, `Attraction`, `DayPlan`, `TripPlan` 등 Pydantic 모델을 정의했다. 프론트엔드에서도 대응하는 TypeScript 타입을 정의해야 한다.

가장 기초적인 `Location` 타입부터 시작해보자. 이것은 경위도 좌표를 나타낸다:

```typescript
// frontend/src/types/index.ts
export interface Location {
  longitude: number
  latitude: number
}
```

이 타입 정의는 백엔드의 Pydantic 모델과 완전히 대응한다. TypeScript에서는 `interface` 키워드를 사용하여 타입을 정의하고, 필드 타입은 콜론으로 구분하며, 기본값은 필요 없다.

다음은 `Attraction` 타입으로, 명소 정보를 나타낸다:

```typescript
export interface Attraction {
  name: string
  address: string
  location: Location
  visit_duration: number
  description: string
  category?: string
  rating?: number
  image_url?: string
  ticket_price?: number
}
```

`Location` 타입을 필드 타입으로 사용하는 중첩 타입이다. 물음표 `?`는 선택적 필드를 나타내며, 백엔드 Pydantic 모델의 `Optional`에 대응한다.

유사하게 `Meal`, `Hotel`, `Budget`, `WeatherInfo` 등의 타입을 정의한다. 최상위 `TripPlan` 타입은 다음과 같다:

```typescript
export interface TripPlan {
  city: string
  start_date: string
  end_date: string
  days: DayPlan[]
  weather_info: WeatherInfo[]
  overall_suggestions: string
  budget?: Budget
}
```

백엔드의 요청 모델에 대응하는 `TripPlanRequest` 타입도 있다:

```typescript
export interface TripPlanRequest {
  city: string
  start_date: string
  end_date: string
  days: number
  preferences: string
  budget: string
  transportation: string
  accommodation: string
}
```

이 타입 정의들은 어떤 용도인가? 첫째, API를 호출할 때 TypeScript가 전달하는 데이터가 `TripPlanRequest` 타입에 맞는지 검사한다. `days`를 문자열로 잘못 입력하면 TypeScript가 즉시 오류를 보고한다. 둘째, API 응답을 받을 때 TypeScript가 응답 데이터가 `TripPlan` 타입에 맞는지 검사한다. 백엔드가 반환하는 데이터 구조가 변경되면 프론트엔드가 즉시 발견할 수 있다. 마지막으로 IDE가 타입 정의를 기반으로 코드 자동 완성을 제공하여, `tripPlan.`을 입력하면 IDE가 자동으로 모든 사용 가능한 필드를 나열한다.

### **13.5.3 API 서비스 캡슐화**

타입 정의가 완료되면 API 호출을 캡슐화할 수 있다. `api.ts` 파일을 생성하여 Axios를 사용해 HTTP 요청을 보낸다:

```typescript
import axios from 'axios'
import type { TripPlanRequest, TripPlan } from '../types'

const api = axios.create({
  baseURL: 'http://localhost:8000/api',
  timeout: 120000, // 2분 타임아웃
  headers: {
    'Content-Type': 'application/json'
  }
})
```

여기서 기본 URL, 타임아웃, 요청 헤더를 설정한 Axios 인스턴스를 생성했다. 왜 타임아웃을 2분으로 설정했는가? 여행 계획을 생성할 때 여러 에이전트를 순서대로 호출해야 하고, 각 에이전트는 LLM과 외부 API를 호출해야 하므로 전체 과정이 10~30초가 걸릴 수 있다. 타임아웃이 너무 짧으면 요청이 중단된다.

다음으로 인터셉터를 추가한다. 인터셉터는 요청 전송 전과 응답 수신 후에 공통 로직을 실행할 수 있다:

```typescript
// 요청 인터셉터
api.interceptors.request.use(
  config => {
    console.log('요청 전송:', config)
    return config
  },
  error => Promise.reject(error)
)

// 응답 인터셉터
api.interceptors.response.use(
  response => {
    console.log('응답 수신:', response)
    return response
  },
  error => {
    console.error('요청 실패:', error)
    return Promise.reject(error)
  }
)
```

마지막으로 API 함수를 정의한다. 이것이 프론트엔드에서 백엔드를 호출하는 유일한 진입점이다:

```typescript
// 여행 계획 생성
export const generateTripPlan = async (request: TripPlanRequest): Promise<TripPlan> => {
  const response = await api.post<TripPlan>('/trip/plan', request)
  return response.data
}
```

이 함수의 타입 시그니처에 주목하자: 매개변수는 `TripPlanRequest` 타입이고, 반환값은 `Promise<TripPlan>` 타입이다. TypeScript가 호출자가 전달하는 매개변수의 요구 충족 여부와 반환값의 올바른 사용 여부를 모두 검사한다.

### **13.5.4 Home 폼 설계**

Home 페이지는 사용자의 진입점으로, 사용자가 여행 요구를 입력하는 폼을 포함한다. Vue 3의 Composition API를 사용하여 코드를 구성한다:

```vue
<script setup lang="ts">
import { ref } from 'vue'
import { useRouter } from 'vue-router'
import { message } from 'ant-design-vue'
import { generateTripPlan } from '@/services/api'
import type { TripPlanRequest } from '@/types'

const router = useRouter()
const loading = ref(false)
const loadingProgress = ref(0)
const loadingStatus = ref('')

const formData = ref<TripPlanRequest>({
  city: '',
  start_date: '',
  end_date: '',
  days: 3,
  preferences: '역사 문화',
  budget: '중간',
  transportation: '대중교통',
  accommodation: '경제형 호텔'
})
</script>
```

여기서 `ref`를 사용하여 반응형 변수를 생성한다. `formData`는 폼 데이터로 타입은 `TripPlanRequest`다. `loading`은 로딩 중인지를, `loadingProgress`는 로딩 진행도를, `loadingStatus`는 로딩 상태 텍스트를 나타낸다.

폼 제출 로직은 다음과 같다:

```typescript
const handleSubmit = async () => {
  loading.value = true
  loadingProgress.value = 0
  
  // 진행 상황 시뮬레이션
  const progressInterval = setInterval(() => {
    if (loadingProgress.value < 90) {
      loadingProgress.value += 10
      if (loadingProgress.value <= 30) loadingStatus.value = '🔍 명소 검색 중...'
      else if (loadingProgress.value <= 50) loadingStatus.value = '🌤️ 날씨 조회 중...'
      else if (loadingProgress.value <= 70) loadingStatus.value = '🏨 호텔 추천 중...'
      else loadingStatus.value = '📋 일정 계획 생성 중...'
    }
  }, 500)
  
  try {
    const response = await generateTripPlan(formData.value)
    clearInterval(progressInterval)
    loadingProgress.value = 100
    router.push({ name: 'result', state: { tripPlan: response } })
  } catch (error) {
    clearInterval(progressInterval)
    message.error('계획 생성 실패, 다시 시도해주세요')
  } finally {
    loading.value = false
  }
}
```

이 코드는 몇 가지 일을 한다. 먼저 `loading`을 true로 설정하여 로딩 상태를 표시한다. 그다음 타이머를 시작하여 500밀리초마다 진행 표시줄과 상태 텍스트를 업데이트한다. 이것은 시뮬레이션된 진행으로, 백엔드의 처리 진행 상황을 정확히 알 수 없기 때문이다. 하지만 사용자에게 시스템이 작동 중임을 알려줄 수 있다.

그런 다음 `generateTripPlan` 함수를 호출하여 API 요청을 보낸다. 이것은 비동기 작업으로 `await`를 사용하여 응답을 기다린다. 요청이 성공하면 타이머를 지우고, 진행도를 100%로 설정한 후 결과 페이지로 이동하며 여행 계획 데이터를 전달한다. 요청이 실패하면 오류 메시지를 표시한다. 마지막으로 성공이든 실패든 `loading`을 false로 설정하여 로딩 상태를 숨긴다.

템플릿 부분은 Ant Design Vue의 컴포넌트를 사용한다:

```vue
<template>
  <div class="home-container">
    <div class="page-header">
      <h1 class="page-title">✈️ 지능형 여행 어시스턴트</h1>
      <p class="page-subtitle">AI 기반 개인화 여행 계획</p>
    </div>
    
    <a-card class="form-card">
      <a-form :model="formData" @finish="handleSubmit">
        <a-form-item label="목적지 도시" name="city" :rules="[{ required: true }]">
          <a-input v-model:value="formData.city" placeholder="예: 베이징" />
        </a-form-item>
        
        <!-- 더 많은 폼 항목... -->
        
        <a-form-item>
          <a-button type="primary" html-type="submit" size="large" :loading="loading">
            계획 시작
          </a-button>
        </a-form-item>
        
        <!-- 로딩 진행 표시줄 -->
        <a-form-item v-if="loading">
          <a-progress :percent="loadingProgress" status="active" />
          <p>{{ loadingStatus }}</p>
        </a-form-item>
      </a-form>
    </a-card>
  </div>
</template>
```

`v-model:value` 지시어는 양방향 데이터 바인딩을 구현한다. 사용자가 입력 상자에 내용을 입력하면 `formData.city`가 자동으로 업데이트되고, `formData.city`의 값이 변경되면 입력 상자의 내용도 자동으로 업데이트된다.

### **13.5.5 Result 페이지 표시**

Result 페이지는 전체 애플리케이션의 핵심으로, 생성된 여행 계획을 표시한다. 이 페이지에는 일정 개요, 예산 내역, 지도 시각화, 일별 일정 세부 내용, 날씨 정보 등 여러 부분이 포함된다.

먼저 지도 시각화다. 고덕 지도 JS API를 사용하여 지도에 명소 위치를 표시한다:

```typescript
import AMapLoader from '@amap/amap-jsapi-loader'

const initMap = async () => {
  const AMap = await AMapLoader.load({
    key: 'your_amap_web_key',
    version: '2.0'
  })
  
  map = new AMap.Map('amap-container', {
    zoom: 12,
    center: [116.397128, 39.916527]
  })
  
  // 명소 마커 추가
  tripPlan.value.days.forEach((day) => {
    day.attractions.forEach((attraction, index) => {
      const marker = new AMap.Marker({
        position: [attraction.location.longitude, attraction.location.latitude],
        title: attraction.name,
        label: { content: `${index + 1}`, direction: 'top' }
      })
      map.add(marker)
    })
  })
}
```

이 코드는 먼저 고덕 지도 SDK를 로드하고, 지도 인스턴스를 생성한 후, 모든 명소를 순회하며 각 명소에 마커(Marker)를 생성한다. 마커의 위치는 명소의 경위도 좌표로, 백엔드 `Attraction` 객체에서 가져온다.

내보내기 기능은 `html2canvas`와 `jsPDF` 라이브러리를 사용한다. `html2canvas`는 DOM 요소를 Canvas로 변환할 수 있고, Canvas를 이미지나 PDF로 내보낼 수 있다:

```typescript
import html2canvas from 'html2canvas'
import jsPDF from 'jspdf'

// 이미지로 내보내기
const exportAsImage = async () => {
  const element = document.getElementById('trip-plan-content')
  const canvas = await html2canvas(element, { scale: 2 })
  const link = document.createElement('a')
  link.download = `${tripPlan.value.city}여행계획.png`
  link.href = canvas.toDataURL()
  link.click()
}

// PDF로 내보내기
const exportAsPDF = async () => {
  const element = document.getElementById('trip-plan-content')
  const canvas = await html2canvas(element, { scale: 2 })
  const imgData = canvas.toDataURL('image/png')
  const pdf = new jsPDF('p', 'mm', 'a4')
  const imgWidth = 210
  const imgHeight = (canvas.height * imgWidth) / canvas.width
  pdf.addImage(imgData, 'PNG', 0, 0, imgWidth, imgHeight)
  pdf.save(`${tripPlan.value.city}여행계획.pdf`)
}
```

이 프론트엔드 기술들을 통해 완전한 웹 애플리케이션을 구현했다. 사용자는 브라우저에서 폼을 작성하고, 요청을 제출하고, AI가 여행 계획을 생성할 때까지 기다리고, 상세한 일정 안배를 확인하고, 지도에서 명소 위치를 보고, 이미지 또는 PDF로 내보낼 수 있다. 전체 과정이 유연하고 자연스럽다. 이것이 현대 웹 애플리케이션의 매력이다.

---

## **13.6 기능 구현 상세 설명**

이 절에서는 지능형 여행 어시스턴트의 핵심 기능 구현을 소개하며, 예산 계산, 로딩 진행 표시줄, 일정 편집, 내보내기 기능, 사이드바 내비게이션이 포함된다.

### **13.6.1 예산 계산 기능**

여행 계획을 세울 때 예산은 매우 중요한 고려 요소다. 사용자는 이번 여행에 대략 얼마가 필요한지, 돈이 어디에 쓰이는지 알아야 한다. 우리의 지능형 여행 어시스턴트는 자동 예산 계산 기능을 제공하며, 비용을 명소 입장료, 호텔 숙박, 식사, 교통의 네 가지 큰 카테고리로 나눈다.

예산 계산 로직은 어디서 구현해야 할까? 우리는 백엔드의 `PlannerAgent`에서 구현하기로 했다. 왜 프론트엔드에서 계산하지 않는가? 예산 추정은 명소의 입장료, 호텔의 가격 범위, 식사 기준 등의 정보를 기반으로 해야 하는데, 이 정보들은 `PlannerAgent`가 일정을 생성할 때 이미 얻은 것들이다. 프론트엔드에서 계산한다면 이 로직을 반복해야 하고 정확하지 않을 수도 있다.

`PlannerAgent`의 프롬프트에서 LLM이 예산 정보를 생성하도록 명확히 요구한다:

```python
PLANNER_AGENT_PROMPT = """
당신은 일정 계획 전문가입니다.

**출력 형식:**
다음 JSON 형식을 엄격히 준수하여 반환하세요:
{
  ...
  "budget": {
    "total_attractions": 180,
    "total_hotels": 1200,
    "total_meals": 480,
    "total_transportation": 200,
    "total": 2060
  }
}

**계획 요구사항:**
...
7. 명소 입장료, 호텔 가격, 식사 기준, 교통 수단에 따라 예산 정보를 추정하여 포함
"""
```

LLM은 일정 내의 명소, 호텔, 식사 안배에 따라 각 항목의 비용을 추정한다. 예를 들어, 일정에 고궁(입장료 60위안), 톈탄(입장료 15위안), 이화원(입장료 30위안)이 포함되어 있다면, 명소 입장료 총비용은 105위안이다. 3일 2박 일정에 경제형 호텔(1박 300위안)이라면, 호텔 총비용은 600위안이다.

프론트엔드에서는 Ant Design Vue의 Statistic 컴포넌트를 사용하여 예산 정보를 표시한다. 이 컴포넌트는 통계 데이터 표시에 특화되어 있으며, 숫자 애니메이션, 접두사/접미사, 커스텀 스타일 등을 지원한다:

```vue
<a-card v-if="tripPlan.budget" title="💰 예산 내역">
  <a-row :gutter="16">
    <a-col :span="6">
      <a-statistic title="명소 입장료" :value="tripPlan.budget.total_attractions" suffix="위안" />
    </a-col>
    <a-col :span="6">
      <a-statistic title="호텔 숙박" :value="tripPlan.budget.total_hotels" suffix="위안" />
    </a-col>
    <a-col :span="6">
      <a-statistic title="식사 비용" :value="tripPlan.budget.total_meals" suffix="위안" />
    </a-col>
    <a-col :span="6">
      <a-statistic title="교통 비용" :value="tripPlan.budget.total_transportation" suffix="위안" />
    </a-col>
  </a-row>
  <a-divider />
  <a-row>
    <a-col :span="24" style="text-align: center;">
      <a-statistic
        title="예상 총비용"
        :value="tripPlan.budget.total"
        suffix="위안"
        :value-style="{ color: '#cf1322', fontSize: '32px', fontWeight: 'bold' }"
      />
    </a-col>
  </a-row>
</a-card>
```

이 코드는 그리드 레이아웃(`a-row`와 `a-col`)을 사용하여 네 항목의 비용을 나란히 표시한다. 각 항목은 `a-statistic` 컴포넌트로 제목과 값을 표시한다. 구분선(`a-divider`) 아래에는 빨간색 큰 글씨로 총비용을 강조 표시한다.

`v-if="tripPlan.budget"` 조건부 렌더링에 주목하자. 예산 정보는 선택적(Pydantic 모델에서 `Optional[Budget]`으로 정의)이므로, LLM이 예산 정보를 생성하지 않은 경우 이 카드가 표시되지 않는다. 이것은 프론트엔드에서의 데이터 오류 처리를 보여준다.

### **13.6.2 로딩 진행 표시줄**

여행 계획 생성은 시간이 많이 걸리는 작업이다. 백엔드는 `AttractionSearchAgent`, `WeatherQueryAgent`, `HotelAgent`, `PlannerAgent`를 순서대로 호출해야 하며, 각 에이전트는 LLM과 외부 API를 호출해야 한다. 전체 과정이 10~30초가 걸릴 수 있다. 사용자가 "계획 시작" 버튼을 클릭한 후 아무런 반응이 없다면, 시스템이 멈춘 것으로 생각하고 페이지를 새로 고침하거나 반복해서 클릭할 수 있다.

사용자 경험을 향상시키기 위해 로딩 진행 표시줄과 상태 메시지를 추가했다. 현재는 진행 상황을 시뮬레이션하지만, 시스템이 작동 중임을 사용자에게 알릴 수 있다.

```typescript
const loading = ref(false)
const loadingProgress = ref(0)
const loadingStatus = ref('')

const handleSubmit = async () => {
  loading.value = true
  loadingProgress.value = 0

  // 진행 상황 업데이트 시뮬레이션
  const progressInterval = setInterval(() => {
    if (loadingProgress.value < 90) {
      loadingProgress.value += 10
      if (loadingProgress.value <= 30) loadingStatus.value = '🔍 명소 검색 중...'
      else if (loadingProgress.value <= 50) loadingStatus.value = '🌤️ 날씨 조회 중...'
      else if (loadingProgress.value <= 70) loadingStatus.value = '🏨 호텔 추천 중...'
      else loadingStatus.value = '📋 일정 계획 생성 중...'
    }
  }, 500)

  try {
    const response = await generateTripPlan(formData.value)
    clearInterval(progressInterval)
    loadingProgress.value = 100
    loadingStatus.value = '✅ 완료!'
    router.push({ name: 'result', state: { tripPlan: response } })
  } catch (error) {
    clearInterval(progressInterval)
    message.error('계획 생성 실패')
  } finally {
    loading.value = false
  }
}
```

### **13.6.3 일정 편집 기능**

AI가 생성한 여행 계획은 매우 지능적이지만, 사용자의 개인적인 요구를 완전히 충족하지 못할 수도 있다. 예를 들어, 특정 명소를 좋아하지 않아 삭제하고 싶거나, 명소의 관람 순서를 조정하고 싶을 수도 있다. 일정 편집 기능을 제공하여 사용자가 일정을 커스터마이징할 수 있게 한다.

편집 기능의 핵심은 **상태 관리**다. 현재 여행 계획과 원래 여행 계획의 두 가지 상태를 유지해야 한다. 사용자가 편집 모드에 진입할 때 원래 계획의 복사본을 저장한다. 편집을 취소하면 원래 계획으로 복원하고, 수정 사항을 저장하면 현재 계획을 업데이트한다:

```typescript
const editMode = ref(false)
const originalPlan = ref<TripPlan | null>(null)

// 편집 모드 진입
const toggleEditMode = () => {
  editMode.value = true
  originalPlan.value = JSON.parse(JSON.stringify(tripPlan.value))
}
```

`JSON.parse(JSON.stringify(...))` 를 사용하여 객체를 깊은 복사한다. 직접 할당하지 않는 이유는? JavaScript에서 객체는 참조 타입이므로, 직접 할당하면 `originalPlan`과 `tripPlan`이 같은 객체를 가리켜 하나를 수정하면 다른 하나에도 영향을 준다. 깊은 복사는 완전히 독립적인 복사본을 만든다.

명소를 이동하는 로직은 배열에서 두 요소의 위치를 교환하는 것이다:

```typescript
// 명소 이동
const moveAttraction = (dayIndex: number, attractionIndex: number, direction: 'up' | 'down') => {
  const attractions = tripPlan.value.days[dayIndex].attractions
  const newIndex = direction === 'up' ? attractionIndex - 1 : attractionIndex + 1
  
  if (newIndex >= 0 && newIndex < attractions.length) {
    [attractions[attractionIndex], attractions[newIndex]] = 
    [attractions[newIndex], attractions[attractionIndex]]
  }
}
```

ES6의 구조 분해 할당 문법으로 두 요소를 교환한다. `[a, b] = [b, a]`는 임시 변수 없이도 우아하게 교환할 수 있는 방법이다.

명소 삭제는 배열의 `splice` 메서드를 사용한다:

```typescript
// 명소 삭제
const deleteAttraction = (dayIndex: number, attractionIndex: number) => {
  tripPlan.value.days[dayIndex].attractions.splice(attractionIndex, 1)
}
```

수정 사항을 저장할 때는 명소 위치가 변경될 수 있으므로 지도를 다시 초기화해야 한다:

```typescript
// 수정 사항 저장
const saveChanges = () => {
  editMode.value = false
  message.success('수정 사항이 저장되었습니다')
  initMap()  // 지도 재초기화
}

// 편집 취소
const cancelEdit = () => {
  if (originalPlan.value) {
    tripPlan.value = originalPlan.value
  }
  editMode.value = false
}
```

템플릿에서는 `editMode`의 값에 따라 다른 UI를 표시한다. 편집 모드에서는 각 명소 옆에 위로 이동, 아래로 이동, 삭제 버튼이 표시된다:

```vue
<div v-if="editMode" class="edit-buttons">
  <a-button size="small" @click="moveAttraction(dayIndex, index, 'up')">위로</a-button>
  <a-button size="small" @click="moveAttraction(dayIndex, index, 'down')">아래로</a-button>
  <a-button size="small" danger @click="deleteAttraction(dayIndex, index)">삭제</a-button>
</div>
```

### **13.6.4 내보내기 기능**

사용자가 만족스러운 여행 계획을 생성한 후, 저장하거나 친구와 공유하고 싶을 수 있다. 이미지와 PDF의 두 가지 내보내기 방식을 제공한다.

내보내기 기능의 핵심은 `html2canvas` 라이브러리다. 이 라이브러리는 DOM 요소를 Canvas로 변환할 수 있고, Canvas를 이미지로 내보낼 수 있다. 하지만 여기에 기술적인 어려움이 있다: 지도는 Canvas로 렌더링되는데, `html2canvas`는 중첩된 Canvas 처리에 호환성 문제가 있다.

지도 Canvas를 이미지로 변환 후 내보내는 등 여러 해결책을 시도했지만, 고덕 지도의 Canvas 렌더링 메커니즘과 크로스 오리진 제한으로 인해 이 방안은 완전히 해결되지 않았다. 실제 프로젝트에서는 다음 대안을 고려할 수 있다:

1. **고덕 지도의 정적 지도 API 사용**: `maps_staticmap` 도구를 호출하여 정적 지도 이미지를 생성하여 동적 지도를 대체
2. **분리 내보내기**: 지도와 일정 내용을 분리하여 내보낸 후 백엔드에서 합치기
3. **스크린샷 서비스 사용**: Puppeteer 등 헤드리스 브라우저를 사용하여 서버 측에서 스크린샷 촬영
4. **내보내기 내용 간소화**: 내보내기 시 지도를 숨기고 텍스트 내용만 내보내기

현재 구현에서는 내보내기 시 지도 부분을 일시적으로 숨기고, 일정의 텍스트 내용과 명소 정보만 내보내는 간소화 방안을 채택했다. 이상적인 방안은 아니지만 내보내기 기능의 사용 가능성을 보장한다.

이미지로 내보내기 로직은 간단하다:

```typescript
import html2canvas from 'html2canvas'

const exportAsImage = async () => {
  const element = document.getElementById('trip-plan-content')
  if (!element) return
  
  const canvas = await html2canvas(element, {
    backgroundColor: '#ffffff',
    scale: 2,
    useCORS: true
  })
  
  const link = document.createElement('a')
  link.download = `${tripPlan.value.city}여행계획.png`
  link.href = canvas.toDataURL('image/png')
  link.click()
  message.success('내보내기 완료!')
}
```

`scale: 2`는 2배 해상도를 사용하여 내보낸 이미지가 더 선명하게 만든다. `useCORS: true`는 크로스 오리진 이미지 로드를 허용하는데, 명소 이미지(Unsplash에서)에 중요하다.

PDF로 내보내기는 추가 단계가 필요하다: Canvas로 변환 후 이미지로 변환하고, 마지막으로 PDF에 추가한다:

```typescript
import jsPDF from 'jspdf'

const exportAsPDF = async () => {
  // 먼저 지도 캡처
  await captureMapImage()
  
  const element = document.getElementById('trip-plan-content')
  if (!element) return
  
  const canvas = await html2canvas(element, {
    backgroundColor: '#ffffff',
    scale: 2,
    useCORS: true,
    allowTaint: true
  })
  
  // 지도 복원
  restoreMap()
  
  const pdf = new jsPDF('p', 'mm', 'a4')
  const imgData = canvas.toDataURL('image/png')
  const imgWidth = 210  // A4 너비
  const imgHeight = (canvas.height * imgWidth) / canvas.width
  
  pdf.addImage(imgData, 'PNG', 0, 0, imgWidth, imgHeight)
  pdf.save(`${tripPlan.value.city}여행계획.pdf`)
  message.success('내보내기 완료!')
}
```

여기서 이미지 높이를 계산하여 종횡비를 유지해야 한다. A4 용지의 너비는 210mm이며, Canvas의 종횡비에 따라 대응하는 높이를 계산한다.

### **13.6.5 사이드바 내비게이션 및 앵커 점프**

Result 페이지의 내용이 많아 일정 개요, 예산 내역, 지도, 일별 일정, 날씨 정보 등이 포함된다. 사용자가 특정 부분으로 빠르게 이동하려면 긴 거리를 스크롤해야 한다. 사이드바 내비게이션과 앵커 점프 기능을 제공하여 사용자가 빠르게 위치를 찾을 수 있게 한다.

사이드바 내비게이션은 Ant Design Vue의 Menu 컴포넌트를 사용한다:

```vue
<a-menu
  v-model:selectedKeys="[activeSection]"
  mode="inline"
  @click="scrollToSection"
>
  <a-menu-item key="overview">📋 일정 개요</a-menu-item>
  <a-menu-item key="budget">💰 예산 내역</a-menu-item>
  <a-menu-item key="map">🗺️ 지도</a-menu-item>
  <a-menu-item key="days">📅 일별 일정</a-menu-item>
  <a-menu-item key="weather">🌤️ 날씨</a-menu-item>
</a-menu>
```

메뉴 항목을 클릭하면 `scrollToSection` 함수가 호출된다:

```typescript
const activeSection = ref('overview')

// 지정 영역으로 스크롤
const scrollToSection = ({ key }: { key: string }) => {
  activeSection.value = key
  const element = document.getElementById(key)
  if (element) {
    element.scrollIntoView({ behavior: 'smooth', block: 'start' })
  }
}
```

`scrollIntoView`는 브라우저 기본 API로, 요소를 가시 영역으로 스크롤한다. `behavior: 'smooth'`는 즉각적인 점프 대신 부드러운 스크롤을 의미한다. `block: 'start'`는 요소의 상단이 가시 영역의 상단에 정렬됨을 의미한다.

페이지의 각 부분에 대응하는 id를 추가해야 한다:

```vue
<div id="overview">
  <!-- 일정 개요 내용 -->
</div>

<div id="budget">
  <!-- 예산 내역 내용 -->
</div>

<div id="map">
  <!-- 지도 내용 -->
</div>
```

이렇게 하면 사용자가 사이드바 내비게이션의 메뉴 항목을 클릭할 때 페이지가 해당 부분으로 부드럽게 스크롤된다.

이 기능들의 구현을 통해, 지능형 여행 어시스턴트는 여행 계획 생성뿐만 아니라 풍부한 상호작용 기능도 제공한다: 예산 계산으로 비용 파악, 로딩 진행 표시줄로 기다리는 불안 해소, 일정 편집으로 개인 요구에 맞는 계획 수립, 내보내기 기능으로 계획 공유 및 저장, 사이드바 내비게이션으로 긴 페이지 탐색 편의. 이 기능들의 조합이 완전하고 사용하기 쉽고 실용적인 웹 애플리케이션을 구성한다.

---

## **13.7 결어**

제13장을 완료한 것을 축하한다!

이 장을 통해 완전한 지능형 여행 어시스턴트 애플리케이션을 구축하는 방법을 배웠을 뿐만 아니라, 더 중요하게는 다음을 습득했다:

1. **시스템 설계 사고**: 복잡한 문제를 여러 개의 간단한 작업으로 분해하는 방법
2. **엔지니어링 실천 능력**: 이론적 지식을 실행 가능한 코드로 전환하는 방법
3. **풀스택 개발 능력**: 프론트엔드와 백엔드 기술 스택을 통합하는 방법
4. **AI 애플리케이션 개발**: LLM을 활용하여 실용적인 애플리케이션을 구축하는 방법

이 프로젝트는 끝이 아니라 시작점이다. 이 프로젝트를 바탕으로 다음을 할 수 있다:

- 더 많은 기능 추가
- 사용자 경험 최적화
- 다른 분야로 확장(예: 지능형 쇼핑 어시스턴트, 지능형 학습 어시스턴트 등)
- 프로덕션 환경에 배포하여 실제 사용자 서비스

가장 좋은 학습 방법은 실천이다. 코드를 읽는 것에 그치지 말고, 직접 수정하고, 확장하고, 최적화해야 한다. 매번의 실천이 다중 에이전트 시스템에 대한 이해를 더욱 깊게 만들 것이다.

AI 애플리케이션 개발의 길에서 더 멀리 나아가길 바란다!