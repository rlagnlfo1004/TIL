## 🔍 AMI(Amazon Machine Image)


AMI는 EC2 **인스턴스를 실행하기 위한 정보**를 모은 단위이다. EC2(가상 컴퓨터)를 실행하기 위해서는 CPU 프로세서 타입, 저장공간, OS 등등의 정보가 필요하다. 이러한 **세팅 정보(템플릿)** 를 저장한 단위라고 생각하면 된다 즉, **AMI는 서버에 필요한 운영체제와 다양한 소프트웨어로 구성된 템플릿**이며, 이러한 템플릿을 AWS에서는 **이미지**라고 부른다.

템플릿에는 컴퓨터의 OS 환경설정 정보 뿐만 아니라, 인스턴스의 EBS에 대한 정보도 모두 포함되어있다. 여기서, EBS에 대한 정보라 함은, EC2 인스턴스가 어떤 EBS 스냅샷과 연결되어있는지에 대한 정보를 의미한다. 또한, **스냅샷**을 기반으로, AMI 구성이 가능하기에, AMI를 사용하여 **현재 상태의 EC2 세팅(템플릿)을 복제해서 다른 계정이나 다른 리전에 전달**도 가능하다.
<br></br>
### ✅ AMI 구성

- 1개 이상의 EBS 스냅샷
- AMI에는 인스턴스가 어떤 EBS 스냅샷과 연결되어있는지에 대한 정보가 포함되어있다.
- 인스턴스 저장 인스턴스의 경우, 루트 볼륨에 대한 템플릿
    
    ex) 운영체제 OS, 애플리케이션 서버
    
- 사용 권한 (어떤 AWS 계정이 사용할 수 있는지)
- EBS 블록 디바이스 맵핑 (EC2 인스턴스를 위한 볼륨 정보는 EBS가 무슨 용량으로 몇개 붙는지)
<br></br>
### ✅ AMI 생성 과정

![Image](https://github.com/user-attachments/assets/89bf0302-3800-4327-ab02-8b54da4e24a5)

출처 : https://inpa.tistory.com/entry/AWS-📚-AMI-Snapshot-개념-백업-사용법-💯-정리?category=947442
<br></br>
1. EBS의 스냅샷을 찍는다.
2. 스냅샷에는 OS, 파일, 시작권한 등이 들어있다.
3. 스냅샷을 S3에 저장한다.
4. 스냅샷을 기반으로 AMI를 만든다.
5. AMI를 가지고 EC2를 실행하거나, 다른 사람에게 공유하거나 복사한다.
<br></br>
### ✅ AMI 카탈로그

AWS Marketplace 카탈로그에는 잘 알려진 판매자가 선별한 오픈 소스 및 상용 소프트웨어가 포함되어 있다.  AMI 카탈로그는 개발 팀을 비롯해 누구나 개발 중인 소프트웨어 또는 프로젝트를 광범위한 심사 없이 등록하거나 교환할 수 있는 커뮤니티 리소스이다. 커뮤니티 AMI 카탈로그에 등록되는 제품은 잘 알려진 판매자의 제품이거나 그렇지 않을 수도 있으며, 일반적으로 추가 조사를 받지 않는다.
<br></br><br></br>
## 🔍 AMI 와 Snapshot의 차이점


기본적으로 **EBS를 백업**한다는 점에서 AMI와 Snapshot은 동일하다.

하지만, AMI는 EC2 인스턴스에 연결되어 있는 모든 **EBS Volume을 동시에 백업**하는 것이라면, Snapshot 기능은 **사용자가 선택한 EBS Volume 하나를 백업**한다는 점에서 차이가 있다. 즉 AMI 이미지 방식은 컴퓨터 한 개를 통째로 백업한다고 생각하면 되지만, 스냅샷은 하드디스크 내용물만 백업하는 것이다.

또한, AMI 와 Snapshot을 이용하여 새로운 **EC2 인스턴스를 생성**할 수 있는데, AMI 인 경우에는 **바로 EC2 인스턴스를 생성**할 수 있고, Snapshot의 경우는 Snapshot 을 이용해 **AMI 를 생성하는 단계를 거쳐야한다**는 차이점이 있다. 스냅샷은 EBS의 내용을 백업한 데이터라 직접 바로 인스턴스를 만드는게 불가능 하다. 
<br></br>
### ✅ 정리

### **AMI**

- EC2에 연결된 전체 볼륨 백업 (EC2 인스턴스에 연결되어있는 OS가 설치된 루트 장치를 포함)
- 인스턴스가 어떤 EBS 스냅샷과 연결되어있는지에 대한 정보도 포함

### SnapShot

- 특점 시점 백업, 특정 EBS 볼륨 백업
<br></br><br></br>
## 📚 참고

[[AWS] 📚 EC2 개념 원리 & 사용 세팅 💯 총정리 (Instance / EBS / AMI)](https://inpa.tistory.com/entry/AWS-%F0%9F%93%9A-EC2-%EA%B0%9C%EB%85%90-%EC%82%AC%EC%9A%A9-%EA%B5%AC%EC%B6%95-%EC%84%B8%ED%8C%85-%F0%9F%92%AF-%EC%A0%95%EB%A6%AC-%EC%9D%B8%EC%8A%A4%ED%84%B4%EC%8A%A4-EBS-AMI)

[[AWS] 📚 AMI / Snapshot 개념 & 백업 사용법 💯 정리](https://inpa.tistory.com/entry/AWS-%F0%9F%93%9A-AMI-Snapshot-%EA%B0%9C%EB%85%90-%EB%B0%B1%EC%97%85-%EC%82%AC%EC%9A%A9%EB%B2%95-%F0%9F%92%AF-%EC%A0%95%EB%A6%AC?category=947442)

[AWS Marketplace의 AMI 기반 제품](https://docs.aws.amazon.com/ko_kr/marketplace/latest/buyerguide/buyer-server-products.html)