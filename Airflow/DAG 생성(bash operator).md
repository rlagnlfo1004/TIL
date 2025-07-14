<img width="480" height="363" alt="Image" src="https://github.com/user-attachments/assets/bef5d8ec-755a-4f58-bbf5-117777e7210a" />

## 0. 오퍼레이터
- **설계도**
- 특정 행위를 할 수 있는 기능을 모아 놓은 클래스
- 즉 파이썬 클래스 파일을 의미한다.
<br><br>
1. Bash 오퍼레이터
    - 리눅스 shell 명령이 실행가능하게 해준다.
    - 즉, 쉘 스크립트 명령을 수행하는 오퍼레이터이다.
2. Python 오퍼레이터
    - python 함수들이 실행가능하게 해준다.
3. S3 오퍼레이터
    - AWS의 object storage인 S3를 컨트롤 하게 해준다.
4. GCS 오퍼레이터
    - GCP의 object storage인 GCS를 다룰 수 있도록 해준다.
<br><br><br><br>
## 1. Task

- 오퍼레이터에서 객체화(인스턴스화) 되어 DAG에서 실행 가능한 오브젝트이다.
- 이들은 방향성을 가지고 연결되며, 순환하지 않는다.
<br><br><br><br>
## 2. Task의 수행 주체

<img width="554" height="299" alt="Image" src="https://github.com/user-attachments/assets/e81d7ce4-b8d3-4fc5-8658-cf416043905c" />
<br><br>
### 스케줄러

1. **DAG Parsing(파싱)**
    - DAG 파일 읽기 → 문법적 이상 없는지, task들의 관계는 어떤지 분석(파싱)
2. **DB에 정보 저장**
3. **DAG 시작시간 결정**
    - 시작 지시
    - scheduler가 직접 DAG를 실행하지 않고, 워커에 지시한다.
<br><br>
### 워커

1. 스케줄러가 시킨 DAG 파일을 찾아 처리한다.
2. 실행 전/후 메타 DB에 업데이트한다.

**즉, 실제 작업 수행의 주체**
<br><br><br><br>
## 3. DAG 생성(bash operator)

<img width="934" height="1169" alt="Image" src="https://github.com/user-attachments/assets/54f7995e-ba38-4451-99f3-30ca3c39a2d7" />
<br><br><br><br>

## 4. docker-compose.yml

<img width="531" height="101" alt="Image" src="https://github.com/user-attachments/assets/9ff64860-5bd2-45a4-90a7-48139f3cc14c" />

- `:`를 기준으로, 왼쪽이 `wsl의 볼륨` , 오른쪽이 `도커 컨테이너의 볼륨` 이다.
- `/dags`는 `docker-compose.yml`의 루트의 디렉토리 위치이다.
- 하지만, 우리가 현재 실제 작업중인 것은 `/wsl/airflow/dags`
- 따라서 이를 수정해주자

<img width="575" height="102" alt="Image" src="https://github.com/user-attachments/assets/1623ea41-2f63-475c-a797-fb8aaecbd837" />
<br><br><br><br>

## 5. 실행

- `docker compose down`  → `docker compose up`

<img width="2048" height="820" alt="Image" src="https://github.com/user-attachments/assets/d6956a3c-be26-4a88-b837-e293c1711a48" />
<br><br>

dag의 첫 번째 task `bash_t1`의 실행 로그를 보면

<img width="1694" height="106" alt="Image" src="https://github.com/user-attachments/assets/7bf9865f-a33f-47d3-ab01-449775370ca8" />

- `echo whoami` → output : `whoami`
<br><br>

dag의 두 번째 task `bash_t2`의 실행 로그를 보면

<img width="1670" height="114" alt="Image" src="https://github.com/user-attachments/assets/75d51601-9fbc-437a-a797-7d64d7219e3a" />

- `echo $HOSTNAME`  → output : `a06c81fce431`
<br><br><br><br>

## 6. `echo $HOSTNAME`

- `a06c81fce431` 는 어디서 나온 것일까?
- docker container list를 보면

<img width="1063" height="404" alt="Image" src="https://github.com/user-attachments/assets/5ff466dd-e823-44d0-a78c-33e223c4207c" />

airflow-airflow-worker-1 이 보인다.

- worker container
- 실제 task를 처리하는 요소는 worker
- 우리는 docker를 통해 airflow를 다운로드 하였기에 worker container가 실제 task를 처리한다.
<br><br><br><br>

## 7. 확인

- docker exec를 통해 실제 컨테이너에 접속하여, 확인해보기

```bash
docker exec -it a06c81fce431 bash
airflow@a06c81fce431:/opt/airflow$ echo $HOSTNAME
```

<img width="486" height="83" alt="Image" src="https://github.com/user-attachments/assets/96db263f-69f6-4313-9e40-8f3ad5db7a86" />