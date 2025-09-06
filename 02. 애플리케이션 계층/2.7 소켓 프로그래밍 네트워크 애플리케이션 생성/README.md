## 2.7 소켓 프로그래밍: 네트워크 애플리케이션 만들기

지금까지 우리는 여러 중요한 인터넷 애플리케이션을 살펴보았습니다. 이제 이러한 네트워크 애플리케이션이 어떻게 개발되는지 알아보겠습니다. <br>
전형적인 인터넷 애플리케이션 서비스는 한 쌍의 프로그램—클라이언트 프로그램과 서버-프로그램으로 구성되며, 이 두 프로그램은 각각 다른 단말 시스템에 위치합니다. <br>
이 두 프로그램이 실행되면 클라이언트 프로세스와 서버 프로세스가 생성되고, 이 두 프로세스는 서로의 소켓(socket)을 통해 읽기와 쓰기를 하며 통신합니다. 따라서 인터넷 애플리케이션을 개발할 때 개발자의 주요 임무는 클라이언트와 서버 프로그램의 코드를 작성하는 것입니다.

인터넷 애플리케이션은 두 가지 유형이 있습니다. 첫 번째 유형은 특정 통신 프로토콜 표준에 따라 동작하는 프로그램입니다. <br>
이러한 프로토콜 표준은 RFC나 기타 표준 문서일 수 있습니다. 이 유형의 애플리케이션은 규칙이 공개되어 있으므로 "오픈(open)" 애플리케이션이라고도 합니다. <br>
이 경우, 클라이언트와 서버 프로그램은 RFC에 정의된 규칙을 반드시 따라야 합니다. 예를 들어, 클라이언트 프로그램이 HTTP 프로토콜 클라이언트의 구현이라면, 그 동작은 RFC 2616에 엄격하게 정의되어 있습니다. <br>
마찬가지로 서버 프로그램도 HTTP 서버의 구현일 수 있으며, 역시 RFC 2616에 정의된 대로 동작해야 합니다. <br>
만약 개발자들이 각각 클라이언트와 서버 프로그램을 작성하면서 RFC의 규칙을 잘 지킨다면, 이 두 프로그램은 서로 통신할 수 있습니다. <br>
실제로 오늘날 많은 인터넷 애플리케이션들은 서로 다른 개발자들이 독립적으로 구현한 클라이언트와 서버가 상호작용하는 구조입니다. 예를 들어, Google Chrome 브라우저가 Apache 웹 서버와 상호작용하거나, BitTorrent 클라이언트가 BitTorrent 트래커와 상호작용하는 경우입니다.

두 번째 유형은 독점 소프트웨어입니다. 이 애플리케이션에서는 클라이언트와 서버 프로그램이 사용하는 애플리케이션 계층 프로토콜이 RFC나 표준 문서에 공개되어 있지 않습니다. <br>
한 명의 개발자(혹은 개발팀)가 클라이언트와 서버 프로그램을 직접 구현하며, 이 개발자가 코드에 대한 완전한 제어권을 가집니다. 하지만 이러한 프로그램은 오픈 프로토콜을 구현하지 않았기 때문에, 다른 독립 개발자는 이 애플리케이션과 상호작용할 수 있는 코드를 작성할 수 없습니다.

이번 절에서는 마스터-슬레이브(주종) 구조 애플리케이션을 개발할 때 가장 중요한 몇 가지 이슈를 깊이 있게 살펴보고, 아주 간단한 마스터-슬레이브 구조 프로그램의 코드를 관찰하며 직접 체험해보겠습니다. <br>
개발 단계에서 개발자가 직면하는 첫 번째 결정은 이 애플리케이션이 TCP를 사용할지 UDP를 사용할지 선택하는 것입니다. <br>
앞서 언급했듯이, TCP는 연결 지향(connection-oriented) 프로토콜로, 두 단말 시스템 간에 신뢰성 있는 바이트 스트림 채널을 제공합니다. <br>
UDP는 비연결(connectionless) 프로토콜로, 각 데이터그램이 독립적으로 전송되며 반드시 도착한다는 보장이 없습니다. <br>
또한, RFC를 구현하는 클라이언트나 서버 프로그램은 잘 알려진 포트 번호를 사용해야 하며, 반대로 독점 소프트웨어를 개발할 때는 이러한 포트 번호를 피해야 합니다.

---

### 2.7.1 UDP 소켓 프로그래밍

이 절에서는 UDP를 사용하는 간단한 마스터-슬레이브 구조 프로그램을 작성합니다. 다음 절에서는 TCP로 유사한 프로그램을 구현합니다.

서로 다른 컴퓨터에서 실행되는 두 프로세스가 통신하려면 상대방의 소켓에 메시지를 전송해야 합니다. <br>
프로세스를 집에, 소켓을 집의 문에 비유할 수 있습니다. 애플리케이션은 문 안쪽에 있고, 전송 계층 프로토콜은 문 바깥쪽에 있습니다. 개발자는 애플리케이션 계층 쪽 소켓을 완전히 제어할 수 있지만, 전송 계층 쪽 소켓은 제어할 수 없습니다.

UDP 소켓을 사용하는 두 프로세스가 어떻게 상호작용하는지 살펴봅시다. <br>
UDP를 사용할 때, 송신자는 데이터를 소켓 밖으로 내보내기 전에 목적지 주소를 반드시 패킷에 붙여야 합니다. <br>
패킷이 송신자의 소켓을 빠져나가면, 인터넷은 이 목적지 주소를 이용해 패킷을 라우팅하여 수신자의 소켓에 전달합니다. <br>
패킷이 수신자의 소켓에 도착하면, 수신 프로세스는 소켓을 통해 패킷을 받아 내용을 확인하고 적절한 동작을 합니다.

그렇다면 패킷에 붙일 목적지 주소는 무엇일까요? 예상대로 대상 호스트의 IP 주소가 포함됩니다. <br>
IP 주소를 붙이면 인터넷의 라우터가 패킷을 올바른 호스트로 전달할 수 있습니다. 하지만 대상 호스트에는 여러 애플리케이션 프로세스가 동시에 실행될 수 있으므로, 특정 소켓을 식별하는 것도 필요합니다. <br>
소켓이 생성될 때마다 컴퓨터는 포트 번호(port number)라는 식별자를 할당합니다. 따라서 목적지 주소에는 소켓의 포트 번호도 포함됩니다. <br>
즉, 송신 프로세스는 패킷에 대상 호스트의 IP 주소와 소켓의 포트 번호를 붙입니다. 또한, 송신자의 출발지 주소(IP 주소와 포트 번호)도 패킷에 붙지만, 이 작업은 일반적으로 운영체제가 자동으로 처리합니다.

아래의 간단한 마스터-슬레이브 구조 애플리케이션 예제를 통해 TCP와 UDP 소켓 프로그래밍이 어떻게 이루어지는지 살펴보겠습니다.

1. 클라이언트는 키보드에서 문자를 읽어 서버로 전송
2. 서버는 받은 문자를 모두 대문자로 변환
3. 서버는 변환된 데이터를 클라이언트로 다시 전송
4. 클라이언트는 변환된 데이터를 화면에 표시

<img width="763" height="638" alt="Image" src="https://github.com/user-attachments/assets/9c28ad3d-e7d5-423e-9887-1bbe4165f394" />

#### UDPClient.py

```python
from socket import *

serverName = 'hostname'
# 서버가 로컬에서 실행될 경우 'localhost' 또는 '127.0.0.1'로 설정해야 합니다.
serverPort = 12000
clientSocket = socket(AF_INET, SOCK_DGRAM)
message = input('Input lowercase sentence:')
clientSocket.sendto(message.encode(), (serverName, serverPort))
modifiedMessage, serverAddress = clientSocket.recvfrom(2048)
print(modifiedMessage.decode())
clientSocket.close()
```

#### UDPServer.py

```python
from socket import *

serverPort = 12000
serverSocket = socket(AF_INET, SOCK_DGRAM)
serverSocket.bind(('', serverPort))
print("The server is ready to receive")
while True:
    message, clientAddress = serverSocket.recvfrom(2048)
    modifiedMessage = message.decode().upper()
    serverSocket.sendto(modifiedMessage.encode(), clientAddress)
```

이 두 프로그램을 테스트하려면 한 컴퓨터에서 UDPClient.py를, 다른 컴퓨터에서 UDPServer.py를 실행하세요. <br>
UDPClient.py에서 서버의 호스트명 또는 IP 주소를 올바르게 설정해야 합니다. 서버에서 UDPServer.py를 실행하면 서버 프로세스가 대기 상태가 되고, 클라이언트가 메시지를 보내면 동작합니다. 클라이언트에서 UDPClient.py를 실행하면 메시지를 입력하고 전송할 수 있습니다.

이 예제를 바탕으로 자신만의 UDP 마스터-슬레이브 구조 애플리케이션을 개발할 수 있습니다. <br>
예를 들어, 서버가 메시지를 대문자로 변환하지 않고 문자열에 's'가 몇 번 등장하는지 세어 그 횟수를 반환하도록 바꿀 수도 있습니다. <br>
또는 클라이언트를 수정하여 사용자가 여러 문장을 계속해서 서버에 보낼 수 있도록 할 수도 있습니다.

---

### 2.7.2 TCP 소켓 프로그래밍

UDP와 달리 TCP는 연결 지향 프로토콜입니다. 즉, 클라이언트와 서버가 메시지를 주고받기 전에 먼저 핸드셰이크(handshake)를 통해 TCP 연결을 설정해야 합니다. <br>
이 TCP 연결의 한쪽 끝은 클라이언트의 소켓에, 다른 쪽 끝은 서버의 소켓에 연결됩니다. TCP 연결을 생성할 때 클라이언트와 서버의 소켓 주소(IP 주소와 포트 번호)가 연결에 연관됩니다. <br>
연결이 설정되면, 어느 쪽이든 데이터를 소켓을 통해 TCP 연결에 넣으면 상대방이 받을 수 있습니다. UDP와 달리, TCP에서는 데이터를 소켓에 넣을 때마다 목적지 주소를 붙일 필요가 없습니다.

<img width="696" height="546" alt="Image" src="https://github.com/user-attachments/assets/b5bc00fd-9bb0-49a7-9f5c-e446d7175d35" /> <br>
<img width="770" height="778" alt="Image" src="https://github.com/user-attachments/assets/6695b612-3d98-4fbf-af20-40624eb5fd76" />

TCP를 사용할 때 클라이언트와 서버가 어떻게 상호작용하는지 살펴봅시다. 클라이언트는 서버와의 초기 연결 작업을 담당합니다. <br>
서버가 클라이언트의 연결 요청에 응답하려면, 서버 프로세스가 미리 실행 중이어야 하며, 외부의 연결 요청을 받을 수 있는 특별한 소켓(문)이 준비되어 있어야 합니다. 클라이언트가 연결 요청을 보내면, 서버는 그 요청을 받아들여 새로운 소켓을 생성하여 해당 클라이언트와의 통신에 사용합니다.

#### TCPClient.py

```python
from socket import *

serverName = 'servername'
serverPort = 12000
clientSocket = socket(AF_INET, SOCK_STREAM)
clientSocket.connect((serverName,serverPort))
sentence = input('Input lowercase sentence:')
clientSocket.send(sentence.encode())
modifiedSentence = clientSocket.recv(1024)
print('From Server: ', modifiedSentence.decode())
clientSocket.close()
```

#### TCPServer.py

```python
from socket import *

serverPort = 12000
serverSocket = socket(AF_INET, SOCK_STREAM)
serverSocket.bind(('', serverPort))
serverSocket.listen(1)
print('The server is ready to receive')
while True:
    connectionSocket, addr = serverSocket.accept()
    sentence = connectionSocket.recv(1024).decode()
    capitalizedSentence = sentence.upper()
    connectionSocket.send(capitalizedSentence.encode())
    connectionSocket.close()
```

이 예제에서는 클라이언트가 서버에 문장을 보내면, 서버는 모든 문자를 대문자로 변환하여 클라이언트에 다시 전송합니다. <br>
TCP 연결을 통해 데이터가 순서대로, 신뢰성 있게 전달됩니다. TCPServer.py에서 서버는 여러 클라이언트의 연결 요청을 계속해서 처리할 수 있습니다.