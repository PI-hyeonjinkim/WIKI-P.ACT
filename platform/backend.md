# 백엔드 — FastAPI

> 출처: memory/platform.md, docs/platform/backend.md (2026-04-22)
> 마지막 갱신: 2026-04-22

---

## 접속

- 로컬: `http://localhost:8080`
- 운영: `https://api.paidevteam.com`

---

## API 목록

### 인증
| 메서드 | 경로 | 설명 |
|--------|------|------|
| POST | `/auth/register` | 회원가입 (organization key 필요) |
| POST | `/auth/login` | 로그인 → JWT |
| POST | `/auth/refresh` | 토큰 갱신 |

- Access Token: 30분 / Refresh Token: 7일
- 라이선스 키: `PIMEDIA-2026-DEMO`, `PIMEDIA-2026-TEST`

### 디바이스
| 메서드 | 경로 | 설명 |
|--------|------|------|
| GET | `/devices` | 목록 |
| POST | `/devices` | 등록 |
| PUT | `/devices/{id}` | 수정 |
| DELETE | `/devices/{id}` | 삭제 |
| POST | `/devices/{id}/command` | MQTT 명령 전송 |
| POST | `/devices/register` | 엣지 자동 등록 (인증 불필요) |

### 기타
- `GET /webrtc/turn-credentials` — TURN 자격증명 발급 (HMAC-SHA1)
- `GET /sites`, `POST /sites` — 사이트 관리
- `/dashboard`, `/logs`, `/models`, `/nvr`, `/proxy`, `/agent`

---

## 주요 파일

| 파일 | 역할 |
|------|------|
| `main.py` | FastAPI 앱, 미들웨어, 라우터 등록 |
| `api/devices.py` | 디바이스 CRUD + register |
| `api/auth.py` | JWT 인증 |
| `api/webrtc.py` | TURN 자격증명 |
| `services/mqtt_consumer.py` | MQTT 수신 → InfluxDB |
| `mcp_server.py` | MCP 서버 (카메라 제어 도구) |

---

## DB 스키마 (devices 주요 컬럼)

```sql
owner_id        — nullable (자동등록 시 NULL, Claim 후 할당)
model           — VARCHAR DEFAULT ''
firmware_version — VARCHAR DEFAULT ''
ip_address      — VARCHAR DEFAULT ''
last_seen       — TIMESTAMP (MQTT 수신마다 갱신)
```

SQLAlchemy `create_all()`로 자동 생성. 별도 마이그레이션 없음.

---

## InfluxDB 버킷

| 버킷 | 보존 | 용도 |
|------|------|------|
| `camera_events` | 무제한 | 진출입 이벤트 |
| `edge_logs` | 90일 | 엣지 MQTT 로그 |
| `service_logs` | 30일 | FastAPI 요청 로그 |
| `container_logs` | 7일 | Docker 컨테이너 로그 |
