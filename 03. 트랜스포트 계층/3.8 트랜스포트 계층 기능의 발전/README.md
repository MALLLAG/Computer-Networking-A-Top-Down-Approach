# 3.8 전송 계층 기능의 진화 (Evolution of Transport-Layer Functionality)

이 장에서는 인터넷 전송 계층의 대표적인 두 프로토콜, **UDP와 TCP**를 중심으로 다루었습니다.  
그러나 지난 수십 년간의 경험을 통해, **이 두 프로토콜만으로는 모든 상황에 완벽히 대응하기 어렵다는 한계**가 드러났습니다.  
이에 따라 전송 계층 기능은 꾸준히 발전해 왔습니다.

## TCP의 발전

지난 10여 년 동안 TCP는 눈부신 진화를 이뤄왔습니다.  
앞서 3.7.1과 3.7.2절에서 살펴본 **클래식 TCP (TCP Tahoe, Reno)** 외에도  
현재는 다양한 새로운 버전들이 개발되어 널리 사용되고 있습니다.

대표적인 TCP 변형:
- **TCP CUBIC**
- **DCTCP (Data Center TCP)**
- **CTCP (Compound TCP)**
- **BBR (Bottleneck Bandwidth and RTT)**

CUBIC(및 그 전신 BIC)은 **TCP Reno보다 훨씬 널리 사용**되고 있으며,  
BBR은 Google의 **B4 내부 네트워크** 및 **공용 웹 서버**에서 배포 중입니다.

이외에도 TCP는 환경에 따라 다양한 형태로 진화했습니다:
- 무선 네트워크용 TCP (높은 RTT, 패킷 재정렬 허용)
- 데이터센터 내부의 초저지연 네트워크용 TCP
- 우선순위를 차등 적용하는 TCP
- 패킷 확인(ACK) 및 세션 종료 방식을 수정한 TCP 등

이제는 더 이상 단일한 “TCP 프로토콜”이라 부르기 어려울 정도로  
다양한 변종이 존재하며, 그 공통점은 **TCP 세그먼트 형식과 혼잡 제어를 공정하게 수행한다는 것**뿐입니다.  
(참고: [Afanasyev 2010], [Narayan 2018])


## QUIC: Quick UDP Internet Connections

TCP와 UDP 모두 특정한 한계를 지닙니다.  
예를 들어, 어떤 애플리케이션은 UDP보다 풍부한 기능이 필요하지만 TCP의 복잡한 연결 과정은 원하지 않을 수 있습니다.  
이런 요구를 해결하기 위해 등장한 것이 **QUIC(Quick UDP Internet Connections)** 입니다.

QUIC은 **UDP 위에서 동작하는 응용 계층(application layer) 프로토콜**로,  
TCP의 신뢰성 있는 전송 기능과 보안을 유지하면서도 훨씬 빠른 연결과 효율적인 통신을 제공합니다.

### QUIC의 개요

- **개발:** Google (Langley 2017)
- **표준화:** IETF에서 진행 중 (RFC QUIC 2020)
- **목표:**
    - HTTP 성능 향상
    - TCP의 비효율적 연결/재전송 문제 해결
    - 브라우저 및 모바일 환경에 최적화

현재 QUIC은 다음에 사용되고 있습니다:
- Google 웹 서버
- Chrome 브라우저
- YouTube 스트리밍 앱
- Android Google Search 앱 등

전 세계 인터넷 트래픽의 약 **7% 이상이 QUIC 기반**으로 동작합니다.

### QUIC의 주요 특징

#### 1. **연결 지향적이고 보안성 강화 (Connection-Oriented & Secure)**

QUIC은 TCP처럼 **연결 기반 프로토콜**이지만,  
연결 수립과 보안 인증 과정을 하나의 단계로 결합했습니다.

- 연결 식별자는 **Source ID / Destination ID** 두 가지입니다.
- 모든 QUIC 패킷은 **암호화되어 전송**됩니다.
- 전송 계층 보안(TLS) 핸드셰이크와 연결 설정이 동시에 수행되어  
  **TCP+TLS보다 훨씬 빠른 연결 성립**이 가능합니다.
- 여러 RTT가 필요한 TCP+TLS보다 훨씬 짧은 시간 내에 통신을 시작할 수 있습니다.

<img width="499" height="182" alt="Image" src="https://github.com/user-attachments/assets/e72ab8c6-0a6c-4f0d-aab6-5f6031620e3c" />

#### 2. **스트림 다중화 (Streams)**

하나의 QUIC 연결 내에서 여러 **스트림(stream)** 을 동시에 처리할 수 있습니다.  
즉, 다수의 HTTP 요청을 병렬로 전송하더라도 **각 스트림이 독립적으로 전송 및 재전송**됩니다.

- 각 연결에는 Connection ID, 각 스트림에는 Stream ID가 존재합니다.
- 스트림 단위의 독립 전송으로 인해 **헤드 오브 라인 블로킹(HOL blocking)** 문제가 사라집니다.
- QUIC은 UDP 기반으로 동작하지만, **TCP 수준의 신뢰성과 순서 보장**을 제공합니다.

이는 과거 SCTP(Stream Control Transmission Protocol)가 시도했던  
“스트림 단위 다중화” 개념을 발전시킨 것입니다.

#### 3. **신뢰성과 혼잡 제어 (Reliable, TCP-friendly Congestion Control)**

<img width="712" height="285" alt="Image" src="https://github.com/user-attachments/assets/35bf182e-f6b5-48e1-94f0-ff4f0d4db71c" />

QUIC은 각 스트림에 대해 **독립적인 신뢰성 보장**을 제공합니다.  
즉, 한 스트림에서 패킷 손실이 발생해도 다른 스트림의 전송에는 영향을 주지 않습니다.

- TCP에서는 모든 HTTP 요청이 하나의 연결로 묶여 전송되므로,  
  하나의 패킷 손실이 전체 요청의 지연을 초래했습니다. (→ HOL Blocking)
- QUIC은 **스트림별 독립 재전송(per-stream retransmission)** 으로 이 문제를 해결했습니다.
- 혼잡 제어는 **TCP Reno(NewReno)** 기반으로 구현되어 있으며,  
  [RFC 5681]과 유사한 ACK 및 윈도우 조정 알고리즘을 사용합니다.

QUIC Draft 명세서([QUIC-Recovery 2020])에서는 다음과 같이 명시되어 있습니다:
> “TCP의 손실 감지 및 혼잡 제어에 익숙한 독자라면, QUIC의 알고리즘도 익숙하게 느낄 것이다.”

### QUIC의 의의

QUIC은 TCP의 기능을 응용 계층으로 옮겨온 형태라고 볼 수 있습니다.  
즉, **애플리케이션 계층 수준에서 신뢰성 있는 데이터 전송과 혼잡 제어를 제공**합니다.

이 말은 곧, QUIC이 **애플리케이션 업데이트 주기(application-update timescale)** 에서  
빠르게 진화할 수 있음을 의미합니다 —  
TCP나 UDP의 커널 단위 업데이트보다 훨씬 빠르게 개선이 가능하다는 뜻입니다.

| 항목 | TCP | QUIC |
|------|------|------|
| 계층 | 전송 계층 | 응용 계층 |
| 기반 프로토콜 | IP | UDP |
| 연결 수립 | 3-way handshake + TLS | 통합된 암호화 핸드셰이크 |
| 다중 스트림 | 불가 (HOL Blocking 발생) | 가능 (스트림별 독립 처리) |
| 혼잡 제어 | AIMD, Reno, Cubic 등 | TCP NewReno 기반 수정 |
| 표준화 | RFC 5681 등 | QUIC RFC (진행 중) |
| 주요 활용 | 일반 인터넷 통신 | HTTP/3, 구글 서비스, YouTube 등 |

QUIC은 전송 계층의 오랜 진화의 결과이자,  
**TCP의 한계를 극복한 차세대 프로토콜**로 자리 잡았습니다.

신뢰성, 혼잡 제어, 보안성을 유지하면서도  
UDP의 민첩성과 확장성을 결합한 것이 핵심입니다.  
이로써 TCP 이후의 전송 계층은 애플리케이션 요구에 맞게 빠르게 진화할 수 있게 되었습니다.
