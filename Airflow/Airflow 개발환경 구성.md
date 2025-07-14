## 0. Airflow란?

- 워크플로우를 만들고 관리할 수 있는 오픈소스 기반 워크플로우 관리 도구
- 파이썬으로 제작된 도구이며, 워크플로우 생성시에도 파이썬을 통해 구현한다.
- 하나의 워크플로우는 DAG(Direcred Acyclic Graph) 이라 부르며, DAG 안에는 1개 이상의 Task가 존재한다.
- Task간 선후행 연결이 가능하되 순환하지 않는 방향성을 가진다.(=DAG)
- Cron 기반의 스케줄링
- 모니터링 및 실패 작업에 대한 재실행이 기능이 가편하다.
<br><br>

즉, Airflow는 워크플로우를 DAG를 사용하여 정의하고, 관리하는 프로그램으로, 자유도가 크며, 확장성이 좋은 워크플로우 관리 프로그램이다.
<br><br><br><br>

## 1. Airflow 특징

### ✅ 장점

- 대규모 워크플로우 환경에서 부하 증가시 수평적 확장 가능한 Kubenetes등의 아키텍처를 지원한다.
- 파이썬에서 지원되는 라이브러리를 활용하여 다양한 도구를 컨트롤 가능하다.(GCP, AWS등 대다수 클라우드에서 제공하는 서비스)
- Airflow에서 제공하는 파이썬 소스 기반으로 원하는 작업을 커스터마이징이 가능하다.(오퍼레이터, Hook, 센서 등)

### ❌ 단점

- 실시간 워크플로우 관리에 적합하지 않다.(최소 분 단위 실행)
- 워크플로우(DAG) 개수가 많아질 경우 모니터링이 쉽지 않다.
<br><br><br><br>

## 2. 개발환경 구성 요구사항

### 도커(Docker)

리눅스내 가상화 관련 커널을 활용하여 어플리케이션을 독립적 환경에서 실행시키는 기술로, VM(가상화 서버) 대비 Guest OS가 없어 경량화 된 가상화 서버로 볼 수 있다.
<br><br>

여러 Airflow 설치 방법이 있지만, 도커 컴포즈를 이용하여 한번에 쉽게 설치가 가능하다.

> **🔍 docker compose란?**
>- 여러 개의 도커 컨테이너 설정을 한방에 관리하기 위한 도커의 확장 기술
>- 에어플로우를 설치하기 위한 도커 컨테이너 세팅 내용이 들어있다.
> 
<br><br>

### 개발환경 플로우 이해

<img width="1406" height="550" alt="Image" src="https://github.com/user-attachments/assets/8430320c-1447-4f6e-9c7d-849d88cd2011" />

- **로컬환경에서 만든 dag 디렉토리를 git을 활용하여 컨테이너까지 배포를 해보고자 한다.**
<br><br>

### Task

- [ ]  로컬 컴퓨터에 파이썬 인터프리터 설치(airflow, 즉 컨테이너와 같은 버전 사용)
- [ ]  IDE, 통합개발 환경을 VS코드를 통해 구성
- [ ]  girhub 레파지토리 생성
- [ ]  로컬 컴퓨터에 Airflow 라이브러리 설치
- [ ]  Git pull이 가능한 환경구성(airflow가 디렉토리 인식)
<br><br><br><br>

## 3. 파이썬 인터프리터 설치

로컬 컴퓨터에 파이썬 인터프리터를 설치해야하는데, 이때는 실행 중인 컨테이너 즉 airflow와 동일한 버전을 사용해야한다. container에서 python 버전을 확인해보자
<br><br>

### container 접속

<img width="1070" height="482" alt="Image" src="https://github.com/user-attachments/assets/e5fe739a-7e27-433e-9824-2c22de79c947" />


- `docker exec` : container에 특정 명령을 실행할 수 있다.
- 이때의 명령을 `/bin/bash` 라고 하면 된다.
- **“container에 접속”** 한다는 것은 container의 shell에 접속하겠다는 의미
- 옵션 `-it`
- 이는 STDIN 표준 입출력을 열고 가상 tty (pseudo-TTY) 를 통해 접속하겠다는 의미이다.
- 즉, `tty`는 터미널 세션을 통해 컨테이너를 조작하겠다는 의미이다.

<img width="423" height="46" alt="Image" src="https://github.com/user-attachments/assets/26a221c0-fc8a-4906-ba31-8e3573c171e7" />

- hostname이 Container id로 바뀐것을 알 수 있다.
- 즉, container 내부에 접속한 상태라는 것이다.
<br><br>

### 파이썬 가상환경의 이해

- 라이브러리 버전 충돌 방지를 위해 설치/사용되는 파이썬 인터프리터 환경을 격리시키는 기술
- 파이썬은 라이브러리 설치 시점에 따라서도 설치되는 버전이 상이한 경우가 많다.

<img width="832" height="160" alt="Image" src="https://github.com/user-attachments/assets/541186b9-0c53-4d8f-918f-2d44ddaa14e3" />

OS 위의 Python은 Gobal 환경의 파이썬이다. C와 D의 프로젝트는 같은 Gobal한 환경의 python을 바라본다고 하자. 이때, C와 D가 같은 라이브러리를 설치하는데, 서로 버전이 다르다면, 어느날 문제가 터진다. C와 D프로젝트는 동일한 파이썬 인터프리터를 바라보고 있기에, 종속성 문제가 발생할 수 있다.

이 문제를 해결하기 위한 방법이 가상환경의 적용이다. Airflow와 B 프로젝트를 서로의 가상환경을 설치하면, 이 둘은 서로 격리 되어있어, 영향을 주지 않고, 각 프로젝트에 필요한 각각의 라이브러리를 각각 설치하여, 이러한 문제를 해결 할 수 있다.

***“프로젝트별, 가상환경 구성하기”*** 
<br><br>

### 파이썬 가상환경 생성

```bash
python3 -m venv ./venv
```

- `venv`
    - 가상환경의 이름
- `./venv`
    - 만들어질 디렉토리 이름
    - 이 안에서 라이브러리가 설치&관리 된다.
<br><br>

### 가상환경 연결

<img width="1198" height="286" alt="Image" src="https://github.com/user-attachments/assets/eb455fd0-18ce-4f43-b51c-b74cec7ec348" />

- Python 인터프리터 선택
<br><br>

<img width="1198" height="356" alt="Image" src="https://github.com/user-attachments/assets/f84b4e6c-d47a-4e33-9000-7ed968195b8b" />

- Python 가상환경을 바라보도록 하기
- global 환경과 venv 중 방금 생성한 venv 선택
<br><br>

<img width="532" height="326" alt="Image" src="https://github.com/user-attachments/assets/e89e2bd3-6f22-43d6-a367-fa23bdcf49e9" />

## 4. Airflow 라이브러리 설치

- 로컬 컴퓨터의 파이썬 가상환경(venv)에 설치를 진행
- Airflow DAG 개발을 위해 Airflow의 라이브러리들이 필요하다.
<br><br>

- 리눅스에서 파이썬 Airflow 라이브러리 설치시 그 자체로 Airflow 서비스 사용가능하다.
- 직접 리눅스에서 pip install 명령으로 Airflow를 설치하지 않는 이유?
    
    → pip install로 Airflow 설치시 저사양의 아키텍처로 설치되며 여러 제약이 존재한다.
    
    → ex. Task를 한번에 1개씩만 실행 가능하다.
    
    **→ 그렇기에 docker를 통해서 설치한다.**
    
<br><br>

### airflow pip install

https://airflow.apache.org/docs/apache-airflow/stable/installation/installing-from-pypi.html

<img width="688" height="45" alt="Image" src="https://github.com/user-attachments/assets/4417c67f-0efe-4a09-af3d-fb29688d1dc8" />

이상태에서 `pip install` 을 하면, 이는 global 환경에 설치하는 것이다.
<br><br>

<img width="1282" height="96" alt="Image" src="https://github.com/user-attachments/assets/455cedcb-ec91-47f5-892c-0677e7f9cc9a" />

interpreter 선택 → venv 환경으로 이동