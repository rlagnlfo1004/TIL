## 🔍 EC2 란?


EC2란 Amazon Elastic Compute Cloud의 줄임말으로서, AWS에서 제공하는 클라우드 컴퓨팅이다. AWS에서 컴퓨터를 대여해주는 서비스이다. 초기 구입비, 세팅비가 전혀 없으며, 그냥 사용한 만큼 비용을 지불하면된다. 또한, EC2는 복잡한 공유기 세팅없이 인터넷을 통해서 자유롭게 접속이 가능하며, 이미지(AMI) 기능도 사용할 수 있다. 이미지(AMI)는 OS의 상태를 그대로 저장하는 기능으로, 윈도우 백업 설정과 같이 새로운 컴퓨터를 만들시에 이미지에 저장된 상태와 똑같은 컴퓨터를 빠르게 생성할 수 있는 것이다.

일반적인 서버의 경우, 컴퓨팅(CPU/RAM), 하드디스크, 랜카드로 나눌 수 있다. 이를 EC2에 대입해 본다면, 다음과 같이 표현 할 수 있다.

- 컴퓨팅 → **인스턴스**
- 하드디스크 → **EBS**
- 랜카드 → **ENI**
<br></br>
## 🔍 인스턴스 이해하기


인스턴스란, 단순히 AWS 클라우드에서 사용하는 **가상의 컴퓨터(서버)**이다. 그중 가상의 컴퓨터에서의 CPU, 메모리, 그래픽 카드 등의 연산을 위한 하드웨어 부분을 담당 하는 것으로, Amazon EC2는 각 사**용 용도에 맞는 다양한 인스턴스 유형**을 제공한다.
<br></br>

### ✅ 인스턴스 유형

Amazon EC2에서는 사용 목적에 따라 다양한 유형을 제공한다. 범용 및 컴퓨팅, 메모리, 저장 최적화 성능 목적에 따라 타입이 여러가지 존재한다. t, m, inf, c, r, g 등의 여러 타입이 존재하며, 특히 t와 m은 범용 타입이기 때문에, 가장 많이 사용하는 프리티어에서 쓰는 타입이다. 주요 스펙들로는 vCPU 수, RAM 크기, EBS 성능, 네트워크 성능 등이 있다.
<br></br>

### ✅ 인스턴스 사이즈

인스턴스의 크기라는 것은 인스턴스 CPU 갯수, 메모리 크기, 성능 등으로 사이즈가 결정되는 것으로, 인스턴스의 사이즈가 클수록 더욱 많은 메모리, CPU, 네트워크 대역폭을 가질 수 있다는 것이다.

예를 들어 **`t2`** 라는 인스턴스 타입에 영단어로 nano, micro, small, large 등의 크기가 나눠져있다. CPU나 메모리를 보면 사이즈가 클 수록 늘어나고, 성능도 빨라진다.

### ✅ 인스턴스 ID

인스턴스 ID는 AWS가 자동 부여하는 고유 식별자이다. 예: `i-0ab1cd23ef456gh78`

### ✅ 인스턴스 라이프 사이클

Amazon EC2 인스턴스는 시작한 순간부터 종료될 때까지 다양한 상태로 전환된다. **EC2의 라이프 사이클이라는 것은 AMI로부터 실행이 되고 나서, 종료될 때까지 EC2가 거치는 일련의 과정을 말한다.**
<br></br>

### ✅ 인스턴스 사이클 상태

![Image](https://github.com/user-attachments/assets/4745ae2a-2be6-43ef-9032-f82e8240606c)

출처 : https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/ec2-instance-lifecycle.html

<br></br>
1. **pending state**
    
    제일 처음 AMI가 실행이 되면 준비상태를 말한다. EC2를 가동하기 위해서 가상머신, ENI, EBS 등이 준비되는 과정으로 시작 중인 상태이다.
    
<br></br>
2. **running state**

    실제로 EC2를 사용할 수 있는 상태 즉 실행 중인 상태를 의미한다. 
    해당 running state에서는 다음 3가지 동작이 가능하다. 
    
    ✔️ **중지**
    
    - 인스턴스를 잠시 멈춰 두는 것으로, 중지 중에는 인스턴스 요금이 청구되지 않는다.
    - 단 EBS 요금, 다른 구성요소(Elastic IP 등)은 청구된다.
    - 중지 후 재시작 할때, **퍼블릭 IP가 변경된다. (프라이빗 IP는 변경되지 않는다. 이를 해결하려면, 탄력적 IP를 사용해야 한다.)**
    - EBS를 사용하는 인스턴스만 중지가 가능하다.
    
    ✔️ **재부팅**
    
    - 인스턴스를 다시 시작하는 것이다.
    - 중지하고 다시 시작과는 달리, **재부팅시 퍼블릭 IP가 변동되지 않는다.**
    
    ✔️ **최대 절전모드**
    
    - 메모리 내용을 보존해서 재시작시 중단지점에서 시작할 수 있는 정지모드이다.
    - 어떤 프로그램을 실행시켰을 때 데이터를 하드디스크에서만 가져오는 것이 아니라 메모리에 올려놓는 것이다.
    - **컴퓨터/노트북의 최대 절전 모드**와 같은 원리라고 볼 수 있다. 프로그램이 켜져있는 상태를 유지하면서 잠시 노트북을 꺼야한다면 최대절전을 한다. 그리고 다시 노트북을 켰을때 아예 OS 재부팅되는게 아니라, 프로그램이 이어서 돌아가게 된다.
    
<br></br>
3. **shutting-down state**
    
    인스턴스를 종료 중인 단계로, 설정에 따라 EBS도 같이 종료 시킬 수 있고, EBS는 남기고 인스턴스만 종료할 수 있다.
    
<br></br>
4. **terminated state**
    
    완전히 종료된 상태로, 인스턴스가 영구적으로 삭제된다.
    
<br></br>
### ✅ 인스턴스 상태 검사

Amazon EC2는 실행 중인 모든 EC2 인스턴스에서 자동 확인하여 문제를 식별한다. 상태 확인이 실패하면 상태 확인에 대한 해당 CloudWatch 지표가 증가하며, CloudWatch 경보가 생성되는데, Amazon EC2는 다음 세가지 상태 확인을 통해 각각 ec2 인스턴스의 상태를 모니터링 한다.

1. **시스템 상태 확인**
    
    **시스템 상태 확인은 인스턴스가 실행되는 AWS 시스템을 모니터링**한다. 즉 **인스턴스가 실행되는 기본 호스트에서의 문제를 탐지**한다. 네트워크, 하드웨어 또는 소프트웨어 문제로 인해 기본 호스트가 응답하지 않거나, 이에 연결할 수 없는 경우 이 상태 확인에 실패한다.
    <br></br>
2. **인스턴스 상태 확인**
    
    인스턴스 상태 검사에서는 개별 인스턴스에 대한 소프트웨어 및 네트워크 연결을 모니터링한다. Amazon EC2는 네트워크 인터페이스(NIC)로 주소 확인 프로토콜(ARP)을 전송하여 인스턴스의 상태를 확인한다. **인스턴스 상태 확인이 실패는 인스턴스의 연결 가능성에 문제**가 있음을 나타낸다. 이 경우, 일반적으로 사용자가 인스턴스를 재부팅하거나 인스턴스 구성을 변경하는 등의 방법으로 문제를 직접 해결해야 한다.
    
    다음은 인스턴스 상태 확인의 실패 원인이 되는 몇 가지 문제의 예 이다.
    
    - 시스템 상태 확인 실패
    - 잘못된 네트워킹 또는 스타트업 구성
    - 메모리가 모두 사용됨
    - 파일 시스템 손상
    - 호환되지 않는 커널
    - 재부팅 중에 인스턴스 상태 확인은 인스턴스를 다시 사용할 수 있을 때까지 실패를 보고한다.
    <br></br>
3. **연결된 EBS 상태 확인**
    
    연결된 EBS 상태 확인은 인스턴스에 연결된 Amazon EBS 볼륨이 연결 가능하고 I/O 작업을 완료할 수 있는지 모니터링한다. 이러한 상태 확인은 컴퓨팅 또는 Amazon EBS 인프라의 근본적인 문제를 감지한다. 연결된 EBS 상태 확인 지표가 실패하면 AWS에서 문제가 해결될 때까지 기다리거나 영향을 받는 볼륨의 교체 또는 인스턴스 중지 후 재시작 등의 조치를 취할 수 있다.
    
<br></br>
### ✅ 인스턴스 경보 설정

Amazon EC2 인스턴스에 서버에서 상태검사가 일부분 오류가 발생해서 서버가 작동 안 했을 때 여러가지 서비스를 연동해서 재부팅하고, 알림을 보내는 설정이 가능하다. Amazon EC2 인스턴스를 생성하면, Cloud Watch를 통해서 경보를 설정할 수 있다. Amazon SNS에서 받아볼 서비스를 구독생성으로 생성이 가능한데, 본인이 받고자 하는 프로토콜을 선택할 수 있다. 보통의 경우 SMS와 이메일로 연동하면 바로바로 쉽게 받아볼 수 있다. 

“원하는 인스턴스를 선택 > 작업 > 모니터링 및 문제 해결 > CloudWatch 경보 관리” 설정을 통해 경보가 발생시 해당 프로토콜로 경보가 발송된다. 이때, 추가로 Amazon Chatbot과 연동하면 슬랙과 연동이 된다. 위와 같은 설정으로 서버에 문제가 생겼을 때, 재부팅 자동화로 알림을 받고 서버를 관리할 수 있는 환경 셋팅이 가능하다.

![Image](https://github.com/user-attachments/assets/1bd120aa-f9a5-433e-bfb3-ae8ed49b0d26)

출처 : https://cloudjs.tistory.com/82#google_vignette
<br></br><br></br>

## 📚 참고

[[AWS] 📚 EC2 개념 원리 & 사용 세팅 💯 총정리 (Instance / EBS / AMI)](https://inpa.tistory.com/entry/AWS-%F0%9F%93%9A-EC2-%EA%B0%9C%EB%85%90-%EC%82%AC%EC%9A%A9-%EA%B5%AC%EC%B6%95-%EC%84%B8%ED%8C%85-%F0%9F%92%AF-%EC%A0%95%EB%A6%AC-%EC%9D%B8%EC%8A%A4%ED%84%B4%EC%8A%A4-EBS-AMI)

[[AWS] 📚 EC2 생명 주기(life cycle) ⏳ 파헤치기](https://inpa.tistory.com/entry/AWS-%F0%9F%93%9A-EC2%EC%9D%98-%EC%83%9D%EB%AA%85-%EC%A3%BC%EA%B8%B0life-cycle-%ED%8C%8C%ED%97%A4%EC%B9%98%EA%B8%B0)

[Amazon EC2 인스턴스 상태 확인 - Amazon Elastic Compute Cloud](https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/monitoring-system-instance-status-check.html#system-status-checks)

[[AWS] AWS EC2인스턴스 경보설정(AWS CloudWatch ↔️ SNS ↔️ Chatbot) / 서비스 연동을 통한 EC2 서버 상태 확인하기](https://cloudjs.tistory.com/82)