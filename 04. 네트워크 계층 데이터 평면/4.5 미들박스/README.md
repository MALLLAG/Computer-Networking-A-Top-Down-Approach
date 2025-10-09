# 4.5 Middleboxes

라우터(Router)는 네트워크 계층에서 핵심적인 역할을 수행하며,  
IP 데이터그램을 목적지로 전달하는 것이 본래의 “핵심 기능(bread and butter job)”이다.  
그러나 현대 네트워크에는 단순 포워딩 외의 기능을 수행하는 **다양한 네트워크 장비(boxes)** 들이 존재한다.  
이러한 장비들을 통칭하여 **Middleboxes(미들박스)** 라고 한다.

## Middlebox의 정의

RFC 3234는 Middlebox를 다음과 같이 정의한다:

> “Any intermediary box performing functions apart from normal,  
> standard functions of an IP router on the data path between a source host and destination host.”

즉, **일반적인 라우터의 포워딩 기능 이외의 작업을 수행하는 중간 장비**를 의미한다.

## Middlebox의 주요 기능 유형

Middlebox들은 주로 세 가지 범주의 서비스를 수행한다.

### NAT Translation
- **NAT(Network Address Translation)** 은 사설 네트워크 주소와 공인 주소 간 변환을 수행.
- 데이터그램의 **IP 주소 및 포트 번호를 재작성(rewriting)** 하여 패킷을 외부 네트워크로 전달.
- 주소 절약 및 내부 네트워크 보호 목적.

### Security Services
- **방화벽(Firewall):** 특정 헤더 필드 값에 따라 트래픽을 허용하거나 차단.
- **DPI (Deep Packet Inspection):** 패킷 내용을 심층 분석하여 의심 트래픽 차단.
- **IDS (Intrusion Detection System):** 공격 패턴을 탐지해 악의적 트래픽 차단.
- **이메일 필터:** 스팸, 피싱 등 유해 메일을 차단하는 응용계층 기반 필터링.

### Performance Enhancement
- **캐싱(Content Caching):** 자주 요청되는 콘텐츠를 로컬에서 저장하여 응답 지연 감소.
- **압축(Compression):** 트래픽 크기를 줄여 전송 효율 향상.
- **로드 밸런싱(Load Balancing):** 서비스 요청을 여러 서버에 분산시켜 효율적 처리.

## Middlebox의 확산과 NFV (Network Function Virtualization)

수많은 Middlebox들이 등장하면서,  
각 장비마다 **별도의 하드웨어, 소프트웨어 스택, 관리 체계**가 필요하게 되었고  
이로 인해 운영비용(OPEX)과 자본비용(CAPEX)이 급격히 증가했다.

이 문제를 해결하기 위한 접근법이 바로 **NFV (Network Function Virtualization)** 이다.

- **NFV 정의:**  
  네트워크 기능을 **전용 하드웨어에서 소프트웨어로 가상화** 하여,  
  범용 하드웨어(서버, 스토리지, 스위치 등) 위에서 실행.
- **핵심 아이디어:**  
  SDN의 "컨트롤/데이터 분리" 개념을 응용하여,  
  미들박스 기능을 소프트웨어 스택으로 통합 운영.
- **결과:**  
  네트워크 장비의 기능(예: 방화벽, NAT, IDS 등)을 클라우드 환경에서 유연하게 배포 및 관리 가능.

## 네트워크 구조의 변화

### 과거의 인터넷 구조
- 네트워크 계층(Network Layer)에는 **라우터만 존재**, 단순 포워딩만 수행.
- 전송 및 응용 계층 기능은 **호스트(Host)** 에서 수행.
- 즉, “**네트워크 코어(core)**”는 단순하고, “**엣지(edge)**”가 지능적이었다.

### 현재의 구조
- Middlebox들은 네트워크 경로 중간에 삽입되어 계층 간 경계를 흐림.
- 예시:
    - NAT: IP/Port 재작성
    - Firewall: L3~L7 헤더 기반 필터링
    - Mail Gateway: 메일 송수신 중간에서 스팸 필터링
- 이러한 장비들은 전통적인 “계층 분리(layer separation)” 개념을 위배하지만,  
  **보안/성능/운영 효율성**이라는 중요한 이유로 계속 사용되고 있다.


## PRINCIPLES IN PRACTICE
### Architectural Principles of the Internet

인터넷은 인류 역사상 가장 복잡하면서도 성공적인 시스템 중 하나이다.  
그 성공의 배경에는 단순하지만 강력한 **아키텍처 원칙**이 존재한다.

RFC 1958 “Architectural Principles of the Internet”에 따르면:

> “The goal is connectivity, and the intelligence is end to end rather than hidden in the network.”

즉,
- **목표는 연결성(connectivity)** 이고,
- **지능(intelligence)은 네트워크 내부가 아니라 끝단(end-to-end)에 존재해야 한다.**

## The IP Hourglass (IP 모래시계 구조)

인터넷은 5계층 구조로 표현되며,  
그중 **IP 계층**은 모든 네트워크 장비에서 반드시 구현되는 **공통 계층**이다.

이 구조는 “**IP Hourglass (IP 모래시계)**” 로 비유된다:

<img width="470" height="313" alt="Image" src="https://github.com/user-attachments/assets/7afb3a28-6351-4630-867c-2e6c97e4fb2d" />


- IP 계층은 “좁은 허리(narrow waist)” 역할을 하며,  
  다양한 하위 기술(Ethernet, WiFi, 5G 등)을 단일 상위 서비스(TCP/UDP)로 연결.
- IP의 단순성과 보편성 덕분에  
  인터넷은 이질적인 물리 네트워크 위에서도 통일된 통신이 가능해졌다.

> Clark (1997): “The IP layer hides differences among link technologies and provides a uniform interface to applications.”

오늘날 중년기에 접어든 인터넷은 Middlebox의 등장으로  
이 “좁은 허리(narrow waist)”가 다소 **넓어지고 있다(widening waist)** 는 관점도 존재한다.

## The End-to-End Argument

RFC 1958의 세 번째 원칙은 다음과 같다:

> “Intelligence is end to end rather than hidden in the network.”

즉, **복잡한 기능은 네트워크 내부가 아니라 종단(endpoints)** 에서 수행해야 한다는 철학이다.  
이 원칙은 Saltzer, Reed, Clark (1984)의 논문에서 정식으로 제시되었다.

### 원문 요약
> 어떤 기능이 통신 시스템 내 여러 위치에서 구현될 수 있을 때,  
> 그 기능이 완전하고 정확하게 수행되려면  
> **응용 계층(즉, 종단)** 에서 구현되어야 한다.
>
> 네트워크 내부에서의 기능 구현은 보조적(성능 개선 목적)일 수 있으나,  
> **기본 기능은 종단(end-to-end)** 에 위치해야 한다.

이 논리를 **“End-to-End Argument”** 라고 부른다.

### 예시: 신뢰성 있는 데이터 전송 (Reliable Data Transfer)
- 네트워크 내부(라우터 등)에서도 일부 오류 제어 수행 가능하지만,
- **완전한 오류 복구는 종단(End Host)에서 TCP가 수행해야 한다.**
- 따라서 **End-to-End 신뢰성**은 네트워크 중심이 아니라  
  **양 끝단에서의 제어**로만 보장될 수 있다.

> “Local error control is incomplete — reliable delivery must be implemented end to end.”

