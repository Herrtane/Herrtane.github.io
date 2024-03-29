---
layout: post
title: <정보처리기사> 02. 소프트웨어 개발 (구현 및 테스트)
date: 2022-03-07 01:11:23 +0900
category: InformationProcessing
comments: true
---

## 개발 지원 도구

- IDE (Integrated Development Environment) : Editor, Compiler, Debugger 등 다양한 툴을 하나의 인터페이스로 통합해 제공하는 도구.
- 빌드 자동화 도구
    - Ant
        - Apache 재단에서 개발한 Java Project의 공식 빌드 자동화 도구
        - XML기반 빌드 스크립트 사용
        - 정해진 표준이 없어 개발자가 모든 것을 정의
        - 스크립트 재사용이 어려움
    - Maven
        - Apache 재단에서 Ant 대안으로 개발
        - 정해진 표준이 존재해 예외사항만 기록됨
        - 컴파일과 빌드 동시 수행 가능
        - 의존성 (Dependency)을 설정하여 라이브러리 관리
    - Gradle
        - Ant와 Maven을 보완해 개발된 도구
        - 안드로이드 스튜디오의 공식 빌드 자동화 도구
        - 의존성 활용
        - Groovy (프로그래밍 언어 중 하나) 기반의 빌드 스크립트 사용
        - 플러그인 설정시 Java, C/C++, Python 언어도 사용 가능
        - 실행할 처리 명령들을 모아 Task단위로 실행
        - 이전에 사용한 Task를 재사용하거나 타인의 Task를 공유하는 등 빌드 캐시 기능 지원 : 빌드 속도 향상
    - Jenkins
        - Java 기반의 오픈소스 형태
        - 서블릿 컨테이너에서 실행되는 서버 기반 도구
        - SVN, Git 등 대부분의 SCM 도구와 연동 가능
        - 친숙한 Web GUI 제공
        - 여러대의 컴퓨터를 이용한 분산 빌드 및 테스트 가능
- 협업 도구 (Groupware)

## 패키징

- 기능 식별 > 모듈화 > 빌드 > 사용자 환경 분석 > 패키징 및 적용 시험 > 패키징 변경 개선 > 배포
- 보안 고려
- 사용자 편의성을 위한 보안성 및 비효율성 문제 고려
- 암호화 알고리즘 적용
- 다양한 이기종 연동 고려

## 릴리즈

- 모듈 식별 > 릴리즈 정보 확인 > 릴리즈 노트 개요 작성 > 영향도 체크 > 정식 릴리즈 노트 작성 > 추가 개선 항목 식별
- 릴리즈 노트를 통해 개선 작업을 공유 및 공지
- 머리말 (Header) : 릴리즈 노트 이름, 소프트웨어 이름, 릴리즈 버전, 릴리즈 날짜, 릴리즈 노트 날짜, 릴리즈 노트 버전 등 포함
- 개요, 목적, 문제요약, 재현항목, 수정내용, 사용자 영향도, SW 지원 영향도, 노트, 면책 조항, 연락처 등 포함

## DRM (Digital Right Management)

- Contents Provider : 저작권자
- Packager : 콘텐츠를 메타데이터와 함께 배포 가능한 형태로 묶어 암호화하는 프로그램
- Contents Distributer : 암호화된 콘텐츠를 유통하는 주체
- Clearing House : 저작권 관리, 라이선스 발급, 결제관리 등을 수행하는 곳
- DRM Controller : 배포된 콘텐츠의 이용 권한을 통제하는 프로그램
- Security Container : 콘텐츠 원본을 안전하게 유통하기 위한 전자적 보안 장치
- Customer : 소비자

## SCM (Software Configuration Management)

- 소프트웨어 개발 과정에서의 변경 사항을 관리하기 위해 개발된 일련의 활동
- 개발 전 단계에 적용되는 활동
- 형상 식별 > 형상 통제 (변경 관리) > 형상 감사 > 형상 기록

### 형상 관리 용어

- import
- check-in = commit
- check-out
- update = pull
- add = add
- merge

### 버전 관리 도구

- 클라이언트/서버 방식 : CVS, SVN(Subversion, CVS의 개선 버전). 중앙 시스템(서버)에 저장되어 관리되는 방식. 서버에 문제가 생기면 작업이 중단됨.
- 분산 저장소 방식 : Git. 하나의 원격 저장소와 분산된 로컬 저장소에 함께 저장되어 관리되는 방식. 원격 저장소에 문제가 생겨도 로컬 저장소의 자료로 작업 재개 가능.
- 공유 폴더 방식 : RCS. 로컬 컴퓨터의 공유 폴더에 저장되어 관리되는 방식. 담당자는 공유 폴더의 파일을 자기 PC로 복사해 컴파일한 후 이상 유무 확인.

## 테스트 과정 및 개요

- 검증(Verification) : 개발자 관점, 명세서를 충족시키는지
    1. 단위 테스트 : 개별 모듈, 서브루틴이 정상적으로 실행되는지 확인. 주로 구조기반 테스트 진행.
    2. 통합 테스트 : 인터페이스 간 시스템이 정상적으로 실행되는지 확인. 상향식 테스트 (Driver) / 하향식 테스트 (Stub) / 혼합식 테스트 (Sandwitch식) / 빅뱅 테스트
    3. 시스템 테스트 : 기능적 테스트 (블랙박스 테스트) / 비기능적 테스트 (화이트박스 테스트)
        - 상향식 테스트 : 하위 모듈들을 Cluster로 결합 > 더미 모듈인 Driver 작성 > Cluster 단위로 테스트 > Cluster는 상위로 이동해 결합, Driver는 실제 모듈로 대체됨.
        - 하향식 테스트 : 주요 제어 모듈의 종속 모듈들을 Stub으로 대체 > 깊이/너비 우선 통합 방식에 따라 Stub들이 하나씩 실제 모듈로 교체 > 모듈 통합마다 테스트 실시 > 새로운 오류가 발생하지 않음을 보증하기 위해 회귀 테스트 실시.
- 확인(Validation) : 사용자 관점, 요구사항을 만족하는지
    1. 인수 테스트 : 알파 테스트 (사용자가 개발자와 함께 통제된 환경에서 진행) / 베타 테스트 (사용자만 통제되지 않은 환경에서 진행) / 형상 테스트
- 정적 테스트 : 프로그램을 실행하지 않고 진행 (워크스루, 인스펙션, 코드검사)
- 동적 테스트 : 프로그램을 실행 (블랙박스, 화이트박스 테스트)

### 테스트의 기본 원리

- 테스팅은 결함이 존재함을 밝히는 것. 결함이 없다고는 증명 불가.
- 완벽한 테스팅은 불가능.
- 개발 초기에 테스팅을 시작하여 기간 단축 및 결함 예방.
- 결함 집중. Pareto 법칙.
- 살충제 패러독스 : 동일한 테스트 케이스의 반복은 새로운 오류를 찾지 못함.
- 테스팅은 정황에 의존적 : 상황에 맞게 테스트 실시.
- 오류-부재의 궤변 : 결함이 없어도 요구사항이 충족되지 않는다면 좋은 소프트웨어가 아님.

## 블랙박스/화이트박스 테스트

### 블랙박스 테스트

- 기능테스트, 명세기반 테스트
- 모듈 내부를 알 수 없음
- 동등 분할 검사 (Equivalence Partitioning Testing) : 입력 조건에 타당한 자료와 타당하지 않은 자료를 유사한 도메인별로 균등하게 그룹핑하여 테스트 케이스를 정하고, 해당 자료에 맞는 결과가 나오는지 확인하는 기법.
- 경계값 분석 (Boundary Value Analysis)
- 원인-효과 그래프 검사 (Cause-Effect Graphing Testing) : 입력 데이터 간의 관계와 출력에 영향을 미치는 상황을 체계적으로 분석한 다음, 효용성이 높은 테스트 케이스를 선정해 검사하는 기법.
- 비교 검사 (Comparison Testing) : 여러 버전의 프로그램에 동일한 자료를 입력하여 동일한 결과가 출력되는지 확인하는 기법.
- 오류 예측 검사 (Error Guessing) : 다른 블랙박스 테스트 기법으로 찾아낼 수 없는 오류를 찾아내는 보충적 검사 기법.

### 화이트박스 테스트

- 경로테스트, 구조기반 테스트
- 모듈 내부를 직접 볼 수 있음
- 소스 코드의 모든 문장을 한번 이상 수행
- 선택, 반복등을 수행함으로서 논리적 경로 점검
- 기초 경로 검사
- 제어 구조 검사 : 조건 검사, 루프 검사, 데이터 흐름 검사

## 테스트 커버리지

- 테스트를 논할 때 테스트가 얼마나 충분한가를 나타낸 것. 즉, 수행한 테스트가 테스트 대상을 얼마나 커버했는지를 나타냄.
- 코드가 실행된다고 해서 모든 오류가 제어된 것은 아니기에, 테스트 커버리지가 100% 완벽한 소프트웨어를 나타내지는 않음. 맹신하지 말것.
- 구문 커버리지 (Statement Coverage) : Line Coverage. 코드 한 줄이 한번 이상 실행되면 충족.
- 조건 커버리지 (Condition Coverage) : 모든 조건식에 해당. 내부 조건 (아래 코드 예시에서 a>0, b<0에 해당)이 true/false의 경우를 충족하는지 확인. 조건 커버리지를 만족하더라도, 내부의 구문 커버리지나 뒤에 나올 결정 커버리지를 만족하지 못하는 경우가 존재. a=1, b=1과 a=-1, b=-1은 조건 커버리지를 만족하나, B의 경우가 실행되지 않아 테스트가 안됨. 
- 결정 커버리지 (Decision Coverage) : Branch Coverage. 모든 조건식이 true/false를 가지게 되면 충족. 아래 코드 예시에서 내부조건이 아닌, 조건식 a>0 && b<0에 해당. 결정 커버리지를 만족하는 테스트를 만드려면, a=1, b=1 과 a=-1, b=-1같이 false만 나오는 식 외에 a=1, b=-1를 넣는 테스트를 만들어야 함.
- 코드 예시

``` cpp
void test (int a, int b) {
    // A
    if (a>0 && b<0){
        // B
    }
    // C
}
```

- 구문 커버리지가 가장 많이 사용됨. 조건, 결정 커버리지는 코드 실행에 대한 테스트보다는 로직 시나리오 테스트에 더 가깝기 때문이고, 조건문이 없는 코드는 아예 커버리지 대상에서 제외하고 테스트를 하지 않기 때문.
- 구문 커버리지를 만족한다면, 비록 모든 시나리오를 테스트한다는 보장은 할 수 없으나, 모든 코드를 테스트 코드가 커버했다고 말할 수 있음.

## 테스트 단위

### 테스트 시나리오

- 테스트 케이스들을 결합

### 테스트 오라클

- 테스트 결과가 올바른지 판단하기 위해 사전에 정의된 참 값을 대입해 비교하는 활동
- True 오라클 : 모든 테스트 케이스의 입력 값에 대해 기대하는 결과를 제공하는 오라클. 중요한 시스템에 주로 사용.
- Sampling 오라클 : 특정한 몇몇 테스트 케이스의 입력 값에 대해서만 기대하는 결과를 제공하는 오라클.
- Heuristic 오라클 : Sampling의 개선. 나머지 입력값들에 대해서는 추정으로 처리하는 오라클.
- 일관성 (Consistent) 검사 오라클 : 변경이 있을 때, 테스트 케이스의 수행 전 후 결과값이 동일한지를 확인하는 오라클.

### 테스트 하네스 (Harness)

- 프로그램을 유닛단위로 테스팅하고 여러가지 조건에 따른 프로그램의 행동과 결과를 모니터링하기 위해 만들어진 테스트 지원 도구
- Test Case, Driver, Stub
- Test Suites : 테스트 케이스들의 집합.
- Test Script : 자동화된 테스트 실행 절차에 대한 명세서.
- Mock Object : 사전에 사용자의 행위를 조건부로 입력해두면, 그 상황에 맞는 예정된 행위를 수행하는 객체.

## 성능 분석

- 처리량 (Throughput), 응답 시간 (Response Time), 경과 시간 (Turn Around Time), 자원 사용률 (Resource Usage)
- 클린 코드를 통해 소스 코드 최적화, 리팩토링과 유지 보수에 유리해짐.
- 소스 코드 품질 분석 도구
    - 정적 분석 도구 : pmd, cppcheck, checkstyle, ccm, cobertuna, SonarQube
    - 동적 분석 도구 : Avalanche, Valgrind

## 모듈 연계

### EAI (Enterprise Application Integration)

- 기업 내 각종 애플리케이션 및 플랫폼 간의 정보 전달, 연계, 통합 등 상호연동이 가능하게 해주는 솔루션
- Point to Point : 변경 및 재사용이 어려움.
- Hub & Spoke : 중앙 집중형 방식. 확장 및 유지보수에 용이하나, Hub에 문제 발생시 시스템 전체 영향.
- Message Bus : ESB 방식. 애플리케이션 사이에 미들웨어를 두어 처리하는 방식. 확장성이 높고 대용량 처리 가능.
- Hybrid : Hub & Spoke와 Message Bus의 혼합 방식. 데이터 병목 현상 최소화.

### ESB (Enterprise Service Bus)

- EAI와 유사하지만, 애플리케이션보다는 서비스 중심의 통합을 지향.
- Coupling을 약하게 유지.
- 관리 및 보안 유지가 쉽고, 높은 수준의 품질 지원이 가능.