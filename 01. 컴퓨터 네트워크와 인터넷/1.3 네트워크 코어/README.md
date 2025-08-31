# 1.3 네트워크 코어

<img width="439" height="357" alt="Image" src="https://github.com/user-attachments/assets/0b791d7d-26c4-4b26-9b21-0e16f139c398" />

현재 우리는 네트워크의 엣지가 어떻게 동작하는지 알게 되었으니, 이제 네트워크의 코어, 즉 패킷 스위치로 이루어진 거대한 망이 네트워크의 각 단말 장치를 연결하는 모습을 살펴볼 수 있습니다. 위 그림에서 파란색 선 부분이 바로 네트워크 코어(Network Core)입니다.

## 1.3.1 패킷 교환(Packet Switching)

네트워크 상의 애플리케이션들은 단말 장치들 간에 메시지(message)를 교환합니다. 이 메시지는 개발자가 원하는 어떤 정보도 담을 수 있으며, 예를 들어 제어 신호(“안녕” 같은)나 데이터(이메일, JPEG 사진, MP3 파일 등)가 될 수 있습니다. <br>
메시지를 한 단말 시스템에서 다른 곳으로 보내기 위해 송신자는 너무 긴 메시지를 여러 개의 작은 조각, 즉 패킷(packet)으로 나눕니다. <br>
패킷은 송수신 장치 사이에서 여러 통신 링크(communication links)와 패킷 스위치(packet switches)를 거칩니다. (현재는 라우터(router)와 링크 계층 스위치(link-layer switch)가 주요 패킷 스위치입니다.) <br>
패킷이 각 통신 링크를 지날 때는 해당 링크의 최대 속도로 전송됩니다. 즉, 송신자 혹은 스위치가 초당 R비트의 속도를 가진 링크로 L비트의 패킷을 보낼 때, 전송에 걸리는 시간은 L/R초입니다.

### 저장 및 전달 전송(Store-and-Forward Transmission)

<img width="490" height="335" alt="Image" src="https://github.com/user-attachments/assets/d38ca652-b562-4602-b743-4908d59f354a" />

대부분의 패킷 스위치는 패킷을 받을 때 저장 및 전달(store-and-forward) 방식을 사용합니다. 즉, 스위치는 전체 패킷을 모두 받은 뒤에야 다음으로 전달할 수 있습니다. <br>
위와 같이 A와 B라는 두 단말 장치가 있고, 그 사이에 라우터 C가 있다고 상상해보세요. 라우터는 일반적으로 여러 입력 링크를 갖지만, 이 예시에서는 각각 한 개의 입력 및 출력 링크만 있습니다. <br>
A가 B에게 3개의 패킷(각각 L비트)을 보내려고 합니다. 어떤 시점에서 A는 1번 패킷의 일부를 이미 보냈고, 그 앞부분은 C에 도착했습니다. 저장 및 전달 방식에서는 라우터가 전체 패킷을 받을 때까지 기다렸다가, 모두 받으면 그제서야 출력 링크로 보내기 시작합니다. <br>
이 방식을 더 잘 이해하기 위해, 패킷 전송이 시작된 순간부터 목적지가 전체 패킷을 받기까지 걸리는 시간을 계산해봅시다. (1.4에서 다룰 전파 지연은 여기서는 무시합니다.)

- A가 전송을 시작한 시점을 0초로 정의합니다.
- 링크 속도가 초당 R비트라면, L/R초 후에 A는 전체 패킷 전송을 마칩니다. 이때 라우터는 전체 패킷을 모두 받았고, 저장합니다.
- L/R초가 되면 라우터는 패킷을 출력 링크로 전송하기 시작합니다.
- 2L/R초가 되면 라우터는 전송을 완료하고, B는 전체 패킷을 받게 됩니다.

즉, 전체 패킷 전송에는 2L/R초의 지연이 발생합니다. <br>
만약 전체 패킷을 다 받지 않고, 비트 하나를 받는 즉시 바로 보내는 식이라면, 전체 지연은 L/R초로 줄어듭니다. 하지만 실제로는 라우터가 패킷 전체를 저장한 뒤, 처리 후 전송해야 하므로 저장 및 전달 방식이 필요합니다. <br>
여러 개의 패킷을 연속으로 보낼 때, 첫 패킷의 전체 지연은 NL/R초(N=링크 수)이고, 이후 각 패킷은 L/R초씩 더 늦게 도착합니다. 즉, 총 지연은 (N+P-1)L/R초(P=패킷 수)입니다.

### 큐잉 지연과 패킷 손실(Queuing Delays and Packet Loss)

각 패킷 스위치는 여러 링크와 연결되어 있으며, 각 링크마다 출력 버퍼(output buffer/queue)가 있습니다. <br>
만약 링크가 다른 패킷을 전송 중이면, 새로 온 패킷은 버퍼에서 대기해야 합니다. 이 큐잉 지연(queuing delay)은 네트워크 혼잡도에 따라 달라집니다. 버퍼가 가득 차면, 새로 온 패킷이나 대기 중인 패킷이 버려질 수 있습니다(패킷 손실, packet loss).

### 포워딩 테이블과 라우팅 프로토콜(Forwarding Tables and Routing Protocols)

라우터는 패킷의 목적지 IP 주소를 보고 포워딩 테이블(forwarding table)을 참조하여 어느 링크로 보낼지 결정합니다. 이 테이블은 자동화된 라우팅 프로토콜에 의해 생성됩니다.

## 1.3.2 회선 교환(Circuit Switching)

<img width="386" height="284" alt="Image" src="https://github.com/user-attachments/assets/e8d9b52f-fa1c-49e5-8d16-998bf5b69d1f" />

패킷 교환과 달리, 회선 교환(circuit switching)은 송신자와 수신자 사이의 전체 경로를 미리 예약하고, 전송 중에는 해당 자원을 독점적으로 사용합니다. 전통적인 전화망이 대표적입니다. <br>
회선 교환 네트워크에서는 데이터 전송 전에 전체 경로가 미리 설정되고, 전송 중에는 그 자원이 계속 할당되어 있습니다. 반면, 패킷 교환은 필요할 때만 자원을 사용하며, 때로는 대기(큐잉)가 필요합니다.

### 회선 교환의 다중화(Multiplexing in Circuit-Switched Networks)

<img width="726" height="223" alt="Image" src="https://github.com/user-attachments/assets/5a346bf2-21b6-4704-839d-fede19285ed3" />

회선 교환에서는 두 가지 다중화 방식이 있습니다.
- **주파수 분할 다중화(FDM):** 하나의 링크를 여러 주파수 대역으로 나누어 각 연결에 할당합니다.
- **시분할 다중화(TDM):** 시간을 여러 슬롯으로 나누어 각 연결에 할당합니다.

예를 들어, 1.536 Mbps의 링크를 24개의 슬롯으로 나누면, 각 연결은 64 kbps의 속도를 가집니다. <br>
회선 교환의 단점은 자원이 비효율적으로 사용된다는 점입니다. 사용자가 대기 중일 때도 자원이 점유되어 낭비됩니다.

## 1.3.3 네트워크의 네트워크(A Network of Networks)

단말 시스템(컴퓨터, 스마트폰 등)은 Access ISP를 통해 인터넷에 연결됩니다. Access ISP끼리도 서로 연결되어야 하며, 이를 통해 '네트워크의 네트워크'가 형성됩니다.

### 네트워크 구조(Network Structure)

1. **Access ISP + 글로벌 트랜짓 ISP**  
   모든 ISP가 하나의 글로벌 트랜짓 ISP에 연결되는 구조입니다.

2. **Access ISP + 여러 글로벌 트랜짓 ISP**  
   여러 트랜짓 ISP가 경쟁하며, Access ISP는 선택할 수 있습니다.

3. **Access ISP + 지역 ISP + Tier-1 ISP**  
   지역 ISP가 Access ISP를 연결하고, Tier-1 ISP가 지역 ISP를 연결합니다.

4. **Access ISP + 지역 ISP + Tier-1 ISP + PoP, 멀티호밍, 피어링, IXP**
    - **PoP(Point of Presence):** ISP가 고객 ISP를 연결하는 지점
    - **멀티호밍(Multi-homing):** ISP가 여러 상위 ISP에 동시에 연결
    - **피어링(Peering):** 동등한 ISP끼리 직접 연결
    - **IXP(Internet Exchange Point):** 여러 ISP가 상호 연결하는 센터

5. **Access ISP + 지역 ISP + Tier-1 ISP + PoP, 멀티호밍, 피어링, IXP + 콘텐츠 제공자 네트워크**  
   구글과 같은 대형 콘텐츠 제공자가 자체 네트워크를 구축하여, 직접 ISP나 IXP에 연결합니다.

<img width="718" height="341" alt="Image" src="https://github.com/user-attachments/assets/894db04f-96e2-462d-8a9b-c6e27b9d6b57" />

