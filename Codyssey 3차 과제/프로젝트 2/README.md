<aside>
📰

**주제:** 도로 안전 뉴스 자동 수집·분류 및 중복 방지

**사용 도구:** Make

</aside>

## 1. 자동화할 반복 업무

토목·건설 분야의 도로 안전 관련 뉴스를 매번 직접 검색하고 분류하여 Notion에 기록하는 업무를 자동화한다.

Google 뉴스 RSS에서 **포트홀·노면 파손**과 **싱크홀·지반 침하** 관련 기사를 가져오고, 제목과 요약의 키워드에 따라 분류한 뒤 Notion의 ‘도로 안전 뉴스 자료’ 데이터베이스에 저장한다. 이미 처리한 기사 URL은 Make Data Store에서 확인하여 중복 등록을 차단한다.

## 2. 자동화 도구 선정 및 선정 이유

**선정 도구: Make**

- RSS와 Notion을 코딩 없이 연결할 수 있다.
- Router와 Filter를 이용해 기사를 주제별로 분류할 수 있다.
- Data Store를 사용해 처리한 기사 URL을 기록하고 중복 생성을 막을 수 있다.
- 실행 결과를 모듈별로 확인하여 오류 지점을 찾기 쉽다.
- 분기 구조가 화면에 시각적으로 나타나 워크플로우를 설명하기 쉽다.
- Make와 Zapier 비교 결과에서 Make가 무료 사용 범위와 분기·확장 편의성 측면에서 더 적합했다.

## 3. 자동화 실행 구조

**전체 흐름**

Google 뉴스 RSS → RSS 기사 감지 → Router → 주제별 Filter → Data Store 중복 검사 → 신규 기사 판별 → 처리 URL 저장 → Notion 등록

| 순서 | 모듈 또는 기능 | 역할 |
| --- | --- | --- |
| 1 | RSS — Watch RSS Feed Items | Google 뉴스 RSS에서 신규 기사 정보를 가져온다. |
| 2 | Router | 가져온 기사를 포트홀 경로와 싱크홀 경로로 분기한다. |
| 3 | Filter | 기사의 Title과 Summary에 지정 키워드가 포함됐는지 확인한다. |
| 4 | Data Store — Check the existence of a record | 기사 URL로 만든 고유 키가 이미 저장되어 있는지 확인한다. |
| 5 | Filter — 새 기사만 | Exists 값이 false인 기사만 다음 단계로 통과시킨다. |
| 6 | Data Store — Add/Replace a Record | 신규 기사의 고유 키를 Data Store에 기록한다. |
| 7 | Notion — Create a Data Source Item | 분류된 기사 정보를 Notion 데이터베이스에 새 항목으로 저장한다. |

## 4. 조건 분기 설계

### 경로 1 — 포트홀·노면 파손

Title과 Summary에 다음 키워드 중 하나가 포함되면 통과한다.

- 포트홀
- 노면 파손
- 도로 파임
- 조건 연결 방식: **OR**

### 경로 2 — 싱크홀·지반 침하

Title과 Summary에 다음 키워드 중 하나가 포함되면 통과한다.

- 싱크홀
- 지반 침하
- 땅꺼짐
- 조건 연결 방식: **OR**

### 분기 우선순위

현재 Router에서 **포트홀·노면 파손 경로가 1번째**, **싱크홀·지반 침하 경로가 2번째**이다. 한 기사가 두 조건을 모두 만족하면 첫 번째 경로에서 고유 키를 먼저 기록하고, 두 번째 경로의 중복 검사에서 차단되도록 구성한다.

## 5. 중복 방지 설계

기사마다 RSS의 URL을 다음 함수로 변환하여 Data Store의 Key로 사용한다.

`md5(RSS의 URL)`

같은 URL은 항상 같은 Key를 생성한다. 각 경로에서 다음 순서로 검사한다.

1. `Check the existence of a record`에서 Key 존재 여부를 확인한다.
2. `Exists = false`인 경우에만 ‘새 기사만’ 필터를 통과시킨다.
3. `Add/Replace a Record`에서 동일한 Key를 저장한다.
4. 이후 Notion에 기사 항목을 생성한다.
5. 다음 실행에서 같은 URL이 들어오면 `Exists = true`가 되어 Notion 생성 단계로 넘어가지 않는다.

> 키워드 필터는 기사의 **분류**를 담당하고, Data Store는 동일 기사의 **중복 저장 방지**를 담당한다.
> 

## 6. Notion 저장 항목

| Notion 속성 | 연결하는 RSS 값 또는 설정 |
| --- | --- |
| 자료명 | Title |
| 분류 | 포트홀·노면 파손 또는 싱크홀·지반 침하 |
| 요약 | stripHTML(Summary) |
| 원문 링크 | URL |
| 출처 | Source Name |
| 발행일 | Date Created |
| 수집일 | Notion 생성 시 자동 기록 |
| 검토 완료 | 기본값 미완료 |
- 관련 자동화 페이지: 도로 안전 뉴스 자동화
- 저장 데이터베이스: 도로 안전 뉴스 자료

## 7. Trigger·Action·조건 분기 설명

- **Trigger:** RSS의 Watch RSS Feed Items가 새 기사 확인을 시작하는 출발점이다.
- **Action 1:** Data Store가 기사 URL의 고유 키 존재 여부를 확인하고 신규 Key를 저장한다.
- **Action 2:** Notion이 분류된 기사를 데이터베이스 항목으로 생성한다.
- **Router:** 하나의 RSS 입력을 두 개의 주제별 경로로 나눈다.
- **Filter:** 주제 키워드 조건과 `Exists = false` 조건을 만족하는 데이터만 다음 단계로 보낸다.

## 8. 구현 화면과 워크플로 흐름

### 8.1 RSS Trigger와 전체 자동 실행 구조

RSS의 **Watch RSS Feed Items**를 시작점으로 두고 Google 뉴스 RSS를 확인하도록 설정하였다. 한 번의 실행 주기에서 가져오는 기사 수는 최대 10건으로 제한하였다. 시나리오가 활성화되면 Make에 설정된 실행 주기에 따라 RSS를 확인하고, 새로 감지된 기사 데이터를 다음 모듈로 전달한다.

!RSS_트리거.svg

전체 시나리오는 RSS 입력을 Router에서 두 경로로 나눈 뒤, 각 경로에서 주제 필터 → 중복 확인 → 신규 기사 판별 → URL Key 저장 → Notion 등록 순으로 실행된다.

!RSS_자동화_구성_화면.svg

**모듈 흐름**

1. RSS에서 기사 Title, Summary, URL, Source Name, Date Created를 가져온다.
2. Router가 기사를 **포트홀·노면 파손** 경로와 **싱크홀·지반 침하** 경로로 분기한다.
3. 각 경로의 키워드 Filter가 관련 기사만 통과시킨다.
4. Data Store가 URL을 해시한 Key의 존재 여부를 확인한다.
5. Key가 없는 새 기사만 다음 단계로 통과시킨다.
6. 새 Key를 Data Store에 저장하고 Notion 데이터베이스 항목을 생성한다.

### 8.2 주제별 키워드 분류

포트홀 경로는 Title과 Summary에서 **포트홀, 도로 파손, 노면 파손** 중 하나를 포함하는지 OR 조건으로 검사한다.

!포트홀_및_노면_파손_필터.svg

싱크홀 경로는 Title과 Summary에서 **싱크홀, 지반 침하, 땅꺼짐** 중 하나를 포함하는지 OR 조건으로 검사한다.

!싱크홀_및_지반_침하_필터.svg

### 8.3 URL 기반 중복 방지

Data Store ‘기사 중복 방지’의 Key에는 기사 URL을 **md5 함수**로 변환한 값을 사용한다. 같은 URL은 같은 Key가 되므로 동일 기사 여부를 일관되게 확인할 수 있다.

!중복_기사_삭제_키_설정.svg

Check the existence of a record의 Exists 값이 false일 때만 ‘새 기사만’ 필터를 통과하도록 설정하였다.

!새_기사만_필터.svg

통과한 기사는 동일한 URL Key를 Data Store에 저장하며, 기존 레코드를 덮어쓰지 않도록 설정하였다. 이후 Notion 생성 모듈이 실행된다.

!신규_기사_URL_키_저장.svg

### 8.4 Notion 데이터베이스 필드 매핑

두 분기 모두 같은 ‘도로 안전 뉴스 자료’ 데이터 소스를 사용하며, 자료명은 Title, 원문 링크는 URL, 발행일은 Date Created, 요약은 stripHTML(Summary), 출처는 Source Name으로 연결하였다. 분류 값만 경로에 따라 다르게 고정하고 검토 완료는 미완료 상태로 저장한다.

**포트홀·노면 파손 경로**

!포트홀_기사_Notion_매핑_1.svg

!포트홀_기사_Notion_매핑_2.svg

**싱크홀·지반 침하 경로**

!싱크홀_기사_Notion_매핑_1.svg

!싱크홀_기사_Notion_매핑_2.svg
