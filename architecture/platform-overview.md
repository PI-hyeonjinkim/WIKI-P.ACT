# 플랫폼 서버 아키텍처

## 서버 접속

- SSH: `ssh lenovo@192.168.33.33` (PW: lenovo)
- API: `https://api.paidevteam.com` (EC2 nginx → ZeroTier → .33:8080)
- Frontend: `http://192.168.33.33:3000`

## 주요 API 엔드포인트

### 디바이스 관리
| 메서드 | 경로 | 설명 |
|--------|------|------|
| POST | `/api/devices/register` | 엣지 자동 등록 (인증 불필요) |
| GET | `/api/devices/unassigned` | 미할당 디바이스 목록 |
| POST | `/api/devices/{id}/claim` | 디바이스 계정 할당 |
| POST | `/api/devices/{id}/ptz` | PTZ 명령 → MQTT 전달 |
| POST | `/api/devices/{id}/reboot` | 재부팅/서비스 재시작 |

### 관리자 (role=admin 필요)
- `/api/admin/users`, `/api/admin/devices`, `/api/admin/stats`

## 인증 구조

- Access Token: 30분
- Refresh Token: 7일
- 401 시 자동 리프레시 + 동시 요청 dedup 처리
- 라이선스 키: `PIMEDIA-2026-DEMO`, `PIMEDIA-2026-TEST`

## 주요 파일 위치

### Backend (`backend/app/`)
| 파일 | 역할 |
|------|------|
| `api/devices.py` | 디바이스 CRUD + register/claim |
| `api/admin.py` | 관리자 API |
| `api/auth.py` | 인증 + 관리자 자동 지정 |
| `services/mqtt_consumer.py` | MQTT 수신 + DB 자동 등록 |
| `config.py` | ADMIN_EMAILS 설정 |
| `main.py` | CORS, 라우터 등록 |

### Frontend (`frontend/src/`)
| 파일 | 역할 |
|------|------|
| `app/(dashboard)/devices/page.tsx` | 디바이스 목록 + Claim |
| `app/(dashboard)/admin/page.tsx` | 관리자 대시보드 |
| `lib/api.ts` | API 클라이언트 (자동 리프레시) |
| `lib/auth-context.tsx` | 로그인/로그아웃 상태 관리 |

## DB 스키마 (devices 테이블)

```sql
owner_id        — nullable (자동등록 시 NULL)
model           — VARCHAR DEFAULT ''
firmware_version — VARCHAR DEFAULT ''
ip_address      — VARCHAR DEFAULT ''
last_seen       — TIMESTAMP (MQTT 수신마다 자동 갱신)
```

## CORS Origins

- `http://localhost:3000`
- `http://192.168.33.33:3000`
- `http://192.168.33.33:8080`
- `https://api.paidevteam.com`

## 알려진 이슈

- PostgreSQL(5432), Redis(6379), InfluxDB(8086) 포트 외부 노출 상태 → 운영 전 방화벽 설정 필요

## 시스템 가이드 문서

전체 아키텍처 가이드: `https://api.paidevteam.com/guide`

> 출처: memory/platform.md (2026-03-14)
