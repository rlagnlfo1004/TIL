## Story COMPUTE-001: 인스턴스 목록 조회


- Keystone 토큰 기반 Nova API 호출
- WebClient 사용 - `GET`
- JSON 응답 → DTO 매핑
- status
    - AWS 스타일로 변경(ex. ACTIVE → running)
        
        
        | AWS 스타일 상태 | Nova 상태 |
        | --- | --- |
        | running | ACTIVE |
        | stopped | SHUTOFF |
        | pending | BUILD |
        | error | ERROR |
    - 실시간 상태의 변경 (WebSocket)
- 페이지네이션 처리(Nova API의 방식)
    - `limit` : 한 번에 가져올 인스턴스 개수
    - `marker` : 이전 요청에서 마지막으로 본 인스턴스의 `id` (다음 페이지 기준점)
<br></br><br></br>
## Story COMPUTE-002: 새 인스턴스 생성

![Image](https://github.com/user-attachments/assets/798915cd-8b89-4b3f-b818-59fe5da08bc0)
<br></br>
1. 이용가능 리소스 조회

| 항목 | 백엔드 API | OpenStack API |
| --- | --- | --- |
| 이미지 목록 | `GlanceService` | `GET /v2/images` |
| Flavor 목록 | `NovaService` | `GET /flavors` |
| 키페어 목록 | `NovaService` | `GET /os-keypairs` |
| 보안 그룹 | `NeutronService` | `GET /v2.0/security-groups` |
| 네트워크 목록 | `NeutronService` | `GET /v2.0/networks` |
1. 사용자 폼 입력

1. 루트 볼륨 생성(선택적)
    - Cinder API 사용: `POST /v3/volumes` → `volume_id` 반환
    
2. Nova 인스턴스 생성 API 호출
    - Nova API: `POST /v2.1/{project_id}/servers`
    - Request :
        - `name`, `imageRef`, `flavorRef`, `key_name`, `networks`, `security_groups`
        - `block_device_mapping_v2` (boot-from-volume 시)
    - 반환값: 인스턴스 ID, 초기 상태 `BUILD`
    
3. 인스턴스 메타데이터 저장
    - 생성 성공 시 아래 정보 DB 저장:
        - 인스턴스 ID, 이름, 상태
        - 이미지/Flavor/KeyPair 정보
        - 네트워크/보안 그룹/볼륨 정보
        - 생성 시간, 유저 정보 등

1. 인스턴스 생성 진행 상황 모니터링 로직
<br></br><br></br>
## Story COMPUTE-003: 인스턴스 제어 (시작/중지/재시작)


1. 인스턴스 제어(Nova 액션 호출)

| 기능 | Nova API | Method | Endpoint |
| --- | --- | --- | --- |
| 시작(Start) | `os-start` | POST | `/servers/{server_id}/action` |
| 중지(Stop) | `os-stop` | POST | `/servers/{server_id}/action` |
| 재시작(Reboot) | `reboot` | POST | `/servers/{server_id}/action` |
| 삭제(Delete) | – | DELETE | `/servers/{server_id}` |
- body:액션을 전달해야 함
    
    ```json
    { "os-stop": null }
    { "os-start": null }
    { "reboot": { "type": "SOFT" } }
    ```
    

2. 상태 변경 실시간 반영
    - 기존 상태 Polling 루프 활용
    - 요청 이후 상태가 `STOPPING`, `STARTING`, `REBOOTING`, `DELETING` 등으로 변경
    - Nova 상태 Polling 후 상태가 바뀌면 → WebSocket으로 프론트에 상태 전송
    
3. 권한 검증 (Developer 이상)
<br></br><br></br>
## Story COMPUTE-004: 키페어 관리


- 사용자가 인스턴스를 SSH로 접속할 수 있도록 **KeyPair 생성/삭제/조회** 지원
- 새 키페어를 생성할 경우 **프라이빗 키를 다운로드**할 수 있어야 함

1. 키페어 생성 (Nova에서 생성)
    - Nova API: `POST /os-keypairs`
    - 응답에 `private_key` 포함됨 (단 한 번만 제공)
    - 생성 후, `private_key`를 사용자에게 다운로드하도록 제공 (`.pem`파일)
    - 키페어 메타데이터 관리 (생성된 키 이름, 생성 시간, userId 등을 DB에 저장)
    - `private_key`는 한 번만 보여줘야 하며 DB에 저장하지 않음

2. 공개키 직접 업로드 (임포트)
    - Nova API: `POST /os-keypairs`
    - Nova는 업로드만 수행, 프라이빗 키는 생성하지 않음

3. 키페어 목록 조회
    - Nova API: `GET /os-keypairs`

4. 키페어 삭제
    - Nova API: `DELETE /os-keypairs/{key_name}`
<br></br><br></br>
## Story COMPUTE-005: 인스턴스 상세 정보 및 모니터링


1. 인스턴스 상세 정보 조회
    - Nova API: `GET` `/v2.1/{project_id}/servers/{server_id}`
    - 포함 항목:
        - 이름, UUID
        - Flavor (CPU/RAM)
        - 이미지 정보
        - 상태 (`status`, `power_state`)
        - IP 주소 (`addresses`)
        - 네트워크 이름 및 포트 ID
        - 볼륨 연결 정보 (`block_device_mapping` → Cinder API 필요)

2. 콘솔 로그 조회
    - Nova API: `GET` `/v2.1/{project_id}/servers/{server_id}/console-output`

3. 기본 메트릭스 (CPU, 메모리 사용률)