## 5.1 제어 평면 소개

네트워크 제어 평면(network control plane)을 이해하기 위해, 이전에 살펴본 포워딩 테이블(forwarding table)과 플로우 테이블(flow table)을 떠올려 보겠습니다.  
이 두 테이블은 네트워크 계층의 **데이터 평면(data plane)** 과 **제어 평면(control plane)** 을 연결하는 핵심 요소입니다.

이 테이블들은 각 라우터의 데이터 평면에서 수행되는 로컬 전달 동작을 정의하며, 일반적인 포워딩에서는 패킷을 특정 출력 포트로 전달할 뿐 아니라,  
패킷을 드롭(drop)하거나 복제(replication)하거나, 혹은 패킷 헤더의 일부를 재작성(rewrite)하는 등의 동작을 포함할 수도 있습니다.

이 장에서는 이러한 포워딩 및 플로우 테이블이 **어떻게 계산되고 유지되며 설치되는지**를 살펴봅니다.  
이를 위해 제어 평면을 구현하는 두 가지 주요 접근 방식을 소개합니다.

### 개별 라우터 제어 (Per-router control)

각 라우터가 독립적으로 라우팅 알고리즘을 실행하는 구조입니다.  
즉, 모든 라우터는 **자체적인 포워딩 기능과 라우팅 기능**을 갖추고 있으며, 서로의 라우팅 정보를 교환하여 포워딩 테이블을 계산합니다.

- 각 라우터는 다른 라우터와 통신하여 포워딩 테이블 값을 계산합니다.
- 이 방식은 인터넷에서 수십 년간 사용되어 왔으며, **OSPF(Open Shortest Path First)** 및 **BGP(Border Gateway Protocol)** 같은 프로토콜이 이 구조를 기반으로 합니다.

<img width="641" height="499" alt="Image" src="https://github.com/user-attachments/assets/0cd30317-fd2a-4e89-a359-9435d0a43150" />

### 논리적으로 중앙집중화된 제어 (Logically centralized control)

이 방식에서는 중앙의 **컨트롤러(controller)** 가 모든 라우터의 포워딩 테이블을 계산하고 분배합니다.  
각 라우터는 **제어 에이전트(Control Agent, CA)** 를 통해 컨트롤러와 통신합니다.

- 각 CA는 컨트롤러의 명령에 따라 동작하며, 자체적으로 포워딩 테이블을 계산하지 않습니다.
- 라우터들은 서로 직접 상호작용하지 않고, 오직 컨트롤러를 통해 제어됩니다.
- 이 구조는 **소프트웨어 정의 네트워킹(SDN, Software-Defined Networking)** 의 핵심 개념입니다.

<img width="639" height="582" alt="Image" src="https://github.com/user-attachments/assets/72cd52fd-7b58-4e97-abf3-11dcb52aa98d" />

### 논리적 중앙 제어의 의미와 실제 사례

“논리적으로 중앙집중화된 제어(logically centralized control)”란 실제 물리적 서버가 여러 대일지라도,  
사용자는 이를 **하나의 중앙 서비스처럼 접근**할 수 있는 구조를 의미합니다.  
이는 **내결함성(fault-tolerance)** 과 **성능 확장성(scalability)** 을 위해 여러 서버로 분산 구현될 수 있습니다.

현대의 대규모 네트워크에서는 이 개념이 실제로 폭넓게 사용되고 있습니다.

- **Google**: B4 글로벌 광역 네트워크(WAN)에서 SDN을 사용하여 데이터센터 간 라우팅 제어
- **Microsoft**: SWAN 프로젝트를 통해 광역 네트워크와 데이터센터 네트워크 간 라우팅을 중앙 컨트롤러로 관리
- **대형 통신사**: COMCAST, Deutsche Telekom 등은 자사 네트워크에 SDN을 통합
- **AT&T**: “SDN은 더 이상 비전이 아니라 현실이다. 내년 말까지 네트워크 기능의 75%가 소프트웨어로 제어될 것이다.”
- **China Telecom / China Unicom**: 데이터센터 내부 및 데이터센터 간 트래픽 모두에 SDN 적용


