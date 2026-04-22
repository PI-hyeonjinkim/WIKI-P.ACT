# 플랫폼 서버 아키텍처

> 출처: memory/platform.md (2026-03-14), docs/platform/ (2026-04-22)
> 마지막 갱신: 2026-04-22

---

## 서버 접속

- SSH: `ssh lenovo@192.168.33.33` (PW: lenovo)
- 프로젝트 경로: `C:/Users/lenovo/Desktop/플랫폼서버`
- API: `https://api.paidevteam.com` (EC2 nginx → ZeroTier → .33:8080)
- Frontend: `https://api.paidevteam.com` / 로컬: `http://192.168.33.33:3000`

---

## docker-compose 서비스 구성

| 서비스 | 포트 | 용도 |
|--------|------|------|
| PostgreSQL 15 | 5432 | 디바이스/사용자/사이트/조직 (DB: ai_camera_platform) |
| Redis | 6379 | 실시간 이벤트 스트림, 캐시 |
| InfluxDB 2.7 | 8086 | 시계열 로그/이벤트 |
| Mosquitto | 1883, 9883(WS) | MQTT 브로커 |
| MinIO | 9000(API), 9001(Console) | 모델/펌웨어/데이터셋 스토리지 |
| MLflow | 5000 | 실험 추적, 모델 레지스트리 |
| FastAPI Backend | 8080→8000 | REST API |
| Next.js Frontend | 3000 | 웹 CMS |
| Prometheus | 9090 | 메트릭 수집 |
| Grafana | 3001 | 대시보드 |
| Fluent Bit | — | Docker 컨테이너 로그 수집 → InfluxDB |

선택 실행: `docker compose --profile gpu up` (Triton), `--profile tools up` (Label Studio)

---

## 백엔드 API 구조 (FastAPI)

### 디바이스 관리
| 메서드 | 경로 | 설명 |
|--------|------|------|
| GET | `/devices` | 디바이스 목록 |
| POST | `/devices` | 디바이스 등록 |
| GET | `/devices/{id}` | 상세 조회 |
| PUT | `/devices/{id}` | 정보 수정 |
| DELETE | `/devices/{id}` | 삭제 |
| POST | `/devices/{id}/command` | MQTT 명령 전송 |
| POST | `/devices/register` | 엣지 자동 등록 (인증 불필요) |

### 인증
| 메서드 | 경로 | 설명 |
|--------|------|------|
| POST | `/auth/register` | 회원가입 (organization key 필요) |
| POST | `/auth/login` | 로그인 → JWT 반환 |
| POST | `/auth/refresh` | 토큰 갱신 |

### 기타
- `GET /webrtc/turn-credentials` — TURN 자격증명 발급
- `GET /sites`, `POST /sites` — 사이트 관리
- `/dashboard`, `/logs`, `/models`, `/nvr`, `/proxy`

---

## 인증 구조

- Access Token: 30분
- Refresh Token: 7일
- 401 시 자동 리프레시 + 동시 요청 dedup 처리
- 라이선스 키: `PIMEDIA-2026-DEMO`, `PIMEDIA-2026-TEST`

---

## InfluxDB 버킷 구조

| 버킷 | 보존 | 용도 |
|------|------|------|
| `camera_events` | 무제한 | 진출입 이벤트 |
| `edge_logs` | 90일 | 엣지 MQTT 로그 |
| `service_logs` | 30일 | FastAPI 요청 로그 |
| `container_logs` | 7일 | Docker 컨테이너 로그 |

---

## 주요 파일 위치

### Backend (`backend/app/`)
| 파일 | 역할 |
|------|------|
| `main.py` | FastAPI 앱 진입점, 미들웨어, 라우터 |
| `api/devices.py` | 디바이스 CRUD + register |
| `api/auth.py` | 인증 + JWT |
| `api/webrtc.py` | TURN 자격증명 발급 |
| `services/mqtt_consumer.py` | MQTT 수신 → InfluxDB |
| `mcp_server.py` | MCP 서버 (카메라 제어 도구) |

### Frontend (`frontend/src/`)
| 파일 | 역할 |
|------|------|
| `components/AIFeatures.tsx` | AI 기능 토글 |
| `components/PTZControl.tsx` | PTZ 조이스틱 UI |
| `lib/api.ts` | API 클라이언트 (자동 리프레시) |

---

## DB 스키마 주요 컬럼 (devices)

```sql
owner_id        — nullable (자동등록 시 NULL, Claim 후 할당)
model           — VARCHAR DEFAULT ''
firmware_version — VARCHAR DEFAULT ''
ip_address      — VARCHAR DEFAULT ''
last_seen       — TIMESTAMP (MQTT 수신마다 자동 갱신)
```

스키마는 SQLAlchemy `create_all()`로 자동 생성 (마이그레이션 파일 없음).

---

## 배포

```bash
# 코드 변경 후
docker compose build backend
docker compose up -d backend

# 전체 재기동
docker compose up -d
```

Claude Code에서: `"백엔드 배포해줘"` 또는 `deploy-platform` 스킬 사용.

---

## 알려진 이슈

- PostgreSQL(5432), Redis(6379), InfluxDB(8086) 포트 외부 노출 상태 → 운영 전 방화벽 설정 필요
