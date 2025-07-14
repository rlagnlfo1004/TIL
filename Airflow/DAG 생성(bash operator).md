![image.png]([attachment:122b01bc-cb53-4f6f-bca1-1cd14bf0d6fe:image.png](https://www.notion.so/image/attachment%3A122b01bc-cb53-4f6f-bca1-1cd14bf0d6fe%3Aimage.png?table=block&id=230a76ff-c9a1-805e-92e9-dc6a0ebc47c8&spaceId=37875086-0463-4536-bf7b-76a3045c93e9&width=2000&userId=86276022-5496-4e5f-ba0e-6d27cf0a31ca&cache=v2))

## 0. 오퍼레이터

- **설계도**
- 특정 행위를 할 수 있는 기능을 모아 놓은 클래스
- 즉 파이썬 클래스 파일을 의미한다.

1. Bash 오퍼레이터
    - 리눅스 shell 명령이 실행가능하게 해준다.
    - 즉, 쉘 스크립트 명령을 수행하는 오퍼레이터이다.
2. Python 오퍼레이터
    - python 함수들이 실행가능하게 해준다.
3. S3 오퍼레이터
    - AWS의 object storage인 S3를 컨트롤 하게 해준다.
4. GCS 오퍼레이터
    - GCP의 object storage인 GCS를 다룰 수 있도록 해준다.

## 1. Task

- 오퍼레이터에서 객체화(인스턴스화) 되어 DAG에서 실행 가능한 오브젝트이다.
- 이들은 방향성을 가지고 연결되며, 순환하지 않는다.

## 2. Task의 수행 주체

![image.png](attachment:9fe9613e-92f2-48b5-9c5a-bdedd5115f76:image.png)

### 스케줄러

1. **DAG Parsing(파싱)**
    - DAG 파일 읽기 → 문법적 이상 없는지, task들의 관계는 어떤지 분석(파싱)
2. **DB에 정보 저장**
3. **DAG 시작시간 결정**
    - 시작 지시
    - scheduler가 직접 DAG를 실행하지 않고, 워커에 지시한다.

### 워커

1. 스케줄러가 시킨 DAG 파일을 찾아 처리한다.
2. 실행 전/후 메타 DB에 업데이트한다.

**즉, 실제 작업 수행의 주체**

## 3. DAG 생성(bash operator)

![image.png](attachment:a9f283e3-2504-40bb-be09-8691a8b11ae8:d58a56eb-727c-4624-afcc-2dcedb7296b4.png)

## 4. docker-compose.yml

![image.png](attachment:2c100ec9-1860-439e-9256-103563533f20:image.png)

- `:`를 기준으로, 왼쪽이 `wsl의 볼륨` , 오른쪽이 `도커 컨테이너의 볼륨` 이다.
- `/dags`는 `docker-compose.yml`의 루트의 디렉토리 위치이다.
- 하지만, 우리가 현재 실제 작업중인 것은 `/wsl/airflow/dags`
- 따라서 이를 수정해주자

![image.png](attachment:b1391d48-aee6-46b8-85bf-51e38d8a37ac:image.png)

## 5. 실행

- `docker compose down`  → `docker compose up`

![image.png](attachment:58841aa3-054e-451e-a9fc-7401054c66b9:image.png)

dag의 첫 번째 task `bash_t1`의 실행 로그를 보면

![image.png](attachment:4d993914-cb26-48af-96f4-e5433bb870e0:image.png)

- `echo whoami` → output : `whoami`

dag의 두 번째 task `bash_t2`의 실행 로그를 보면

![image.png](attachment:4e88113d-18ce-401d-a7f9-807c82961f57:image.png)

- `echo $HOSTNAME`  → output : `a06c81fce431`

## 6. `echo $HOSTNAME`

- `a06c81fce431` 는 어디서 나온 것일까?
- docker container list를 보면

![image.png](attachment:2605981f-8038-434b-9880-b57dd17bafcd:image.png)

airflow-airflow-worker-1 이 보인다.

- worker container
- 실제 task를 처리하는 요소는 worker
- 우리는 docker를 통해 airflow를 다운로드 하였기에 worker container가 실제 task를 처리한다.

## 7. 확인

- docker exec를 통해 실제 컨테이너에 접속하여, 확인해보기

```bash
docker exec -it a06c81fce431 bash
airflow@a06c81fce431:/opt/airflow$ echo $HOSTNAME
```

![image.png](attachment:551efc1e-e234-4438-bfa0-863c3716d1a4:image.png)