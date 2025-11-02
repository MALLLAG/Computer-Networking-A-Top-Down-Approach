## 5.6 ICMP: 인터넷 제어 메시지 프로토콜 (Internet Control Message Protocol)

**인터넷 제어 메시지 프로토콜(ICMP)**은 [RFC 792]에 명시되어 있으며, 호스트와 라우터가 네트워크 계층 정보를 서로 주고받기 위해 사용됩니다.  
가장 일반적인 사용 목적은 **오류 보고(error reporting)** 입니다.  
예를 들어, HTTP 세션 중 “Destination network unreachable(대상 네트워크에 도달할 수 없음)”이라는 오류 메시지를 본 적이 있을 수 있습니다. 이 메시지는 바로 ICMP에서 비롯된 것입니다.  
라우터가 특정 호스트로의 경로를 찾지 못했을 때, 해당 라우터는 송신 호스트로 ICMP 메시지를 보내 오류를 알립니다.

ICMP는 **IP의 일부**로 간주되지만, **구조적으로는 IP 바로 위에 위치**합니다.  
즉, ICMP 메시지는 IP 데이터그램 내에서 전송됩니다(IP 페이로드로 포함됨).  
수신 측에서는 IP 데이터그램의 상위 계층 프로토콜 번호가 1(ICMP를 의미)로 지정되어 있으면, 해당 데이터그램을 ICMP로 처리합니다.

ICMP 메시지는 **유형(Type)** 과 **코드(Code)** 필드를 가지며, 오류를 유발한 IP 데이터그램의 헤더와 처음 8바이트를 포함합니다.  
이 덕분에 송신자는 어떤 데이터그램에서 오류가 발생했는지 확인할 수 있습니다.  
ICMP는 단순히 오류 보고뿐 아니라 여러 네트워크 진단 기능에도 사용됩니다.

### ICMP의 대표적인 예: Ping 프로그램

잘 알려진 **ping** 프로그램은 ICMP **type 8, code 0** (echo request)을 전송합니다.  
목적지 호스트는 이 요청을 받으면 **type 0, code 0** (echo reply) 메시지를 보냅니다.  
이 과정을 통해 네트워크 연결 여부와 응답 시간을 확인할 수 있습니다.

운영체제는 이러한 ping 요청 및 응답을 내부적으로 처리하며, 서버가 별도의 프로세스로 작동하는 것은 아닙니다.  
Ping 프로그램은 OS에 ICMP 메시지 생성을 요청해야 합니다.

### 기타 ICMP 메시지 유형

| ICMP Type | Code | 설명 |
|------------|------|------|
| 0 | 0 | Echo reply (ping 응답) |
| 3 | 0 | Destination network unreachable (네트워크 도달 불가) |
| 3 | 1 | Destination host unreachable (호스트 도달 불가) |
| 3 | 2 | Destination protocol unreachable (프로토콜 도달 불가) |
| 3 | 3 | Destination port unreachable (포트 도달 불가) |
| 3 | 6 | Destination network unknown (알 수 없는 네트워크) |
| 3 | 7 | Destination host unknown (알 수 없는 호스트) |
| 4 | 0 | Source quench (혼잡 제어) |
| 8 | 0 | Echo request (ping 요청) |
| 9 | 0 | Router advertisement (라우터 광고) |
| 10 | 0 | Router discovery (라우터 탐색) |
| 11 | 0 | TTL expired (TTL 만료) |
| 12 | 0 | IP header bad (IP 헤더 오류) |

### ICMP와 Traceroute 프로그램

**Traceroute**는 호스트에서 목적지까지의 경로를 추적하기 위한 도구입니다.  
이 프로그램은 **ICMP 메시지를 기반**으로 동작합니다.  
Traceroute는 여러 개의 IP 데이터그램을 보내며, 각 데이터그램의 **TTL(Time To Live)** 값을 1씩 증가시킵니다.

- TTL이 만료되면, 해당 라우터는 **ICMP Type 11 (TTL expired)** 메시지를 송신자에게 반환합니다.
- 이 메시지를 통해 송신자는 해당 라우터의 IP 주소와 이름을 알아낼 수 있습니다.
- 목적지 호스트에 도달하면, **ICMP Type 3, Code 3 (port unreachable)** 메시지가 반환되어 경로 추적이 종료됩니다.

Traceroute는 일반적으로 TTL별로 **3개의 패킷**을 전송하며, 각 TTL 구간에 대해 **3개의 응답 결과**를 제공합니다.

### ICMPv6 (IPv6용 ICMP)

IPv6에서는 새로운 버전의 ICMP가 **RFC 4443**에서 정의되었습니다.  
이는 기존 ICMP 메시지를 재구성하고, IPv6의 새로운 기능을 위해 **새로운 타입과 코드**를 추가했습니다.  
예를 들어, “Packet Too Big” 메시지나 “Unrecognized IPv6 Options” 오류 코드가 이에 해당합니다.

