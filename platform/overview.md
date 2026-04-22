# 플랫폼 전체 구조

> 출처: raw/decisions/2026-04-17_platform-architecture.md, docs/platform/ (2026-04-22)
> 마지막 갱신: 2026-04-22

---

## 제품 구성

**P.ACT (MK3pro)** — 엣지 AI 카메라
- Hailo-8L NPU, RPi CM5 (aarch64)
- 현장 AI 추론 (피플카운팅, 객체감지, PTZ 자동 추적)

**플랫폼** — 통합 관리 서버
- NVR CMS형 웹 서비스
- 멀티 디바이스 등록/관제, 펌웨어 배포, 모델 관리

---

## 전체 구조도

```
[브라우저 / NVR]
    │ HTTPS / WSS / ONVIF
    ▼
[EC2: api.paidevteam.com (54.253.91.60)]
    │ nginx 리버스 프록시 + coturn TURN (3478)
    ▼
[ZeroTier 10.235.62.x]
    │
    ├── [플랫폼 서버 192.168.33.33]
    │       FastAPI / Next.js / PostgreSQL
    │       Redis / InfluxDB / Mosquitto
    │       MinIO / MLflow / Grafana
    │
    └── [엣지 디바이스 (ZeroTier)]
            edge-settings / edge-ptz / mediamtx / Hailo

[로컬 망 디바이스 192.168.33.x]
    └── 직접 접근 가능 (ZeroTier 불필요)
```

---

## 플랫폼 서비스 목록

| 서비스 | 포트 | 역할 |
|--------|------|------|
| FastAPI backend | 8080→8000 | REST API, MQTT consumer |
| Next.js frontend | 3000 | 웹 CMS |
| PostgreSQL 15 | 5432 | 디바이스/사용자/사이트/조직 |
| Redis | 6379 | 온라인 상태(TTL 120s), PTZ 캐시 |
| InfluxDB 2.7 | 8086 | 시계열 로그 |
| Mosquitto | 1883, 9883(WS) | MQTT 브로커 |
| MinIO | 9000, 9001 | 펌웨어/모델 스토리지 |
| MLflow | 5000 | 모델 실험 추적 |
| Grafana | 3001 | 대시보드 |
| Prometheus | 9090 | 메트릭 수집 |

---

## 플랫폼 서버 관리

```bash
# 컨테이너 상태
docker ps

# 로그 확인
docker logs platform-backend --tail 50

# 재빌드 배포
docker compose up -d --build backend

# 전체 재시작
docker compose up -d
```

디바이스 온라인 상태:
```bash
docker exec platform-redis redis-cli get device:status:MK3-0f5d86bc
# 값 있으면 online (TTL 120초)
```
