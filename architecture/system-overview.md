# 시스템 전체 구조

> 출처: raw/decisions/2026-04-17_platform-architecture.md
> 마지막 갱신: 2026-04-17

---

## 제품 두 가지

**P.ACT (MK3pro)** — 엣지 AI 카메라
- Hailo-8 NPU 탑재, ARM aarch64
- 클라우드 없이 현장에서 직접 AI 추론
- 기능: 객체 감지, 피플카운팅, PTZ 제어, ONVIF, WebRTC 스트리밍, MQTT 통신

**플랫폼** — 통합 관리 서비스
- NVR CMS형 웹 서비스
- 멀티 디바이스 등록·관제, 펌웨어 배포, 모델 관리
- 서버: 192.168.33.33, Docker Compose

---

## 전체 구조도

```
[브라우저]
    │ HTTPS / WSS
    ▼
[EC2: api.paidevteam.com (54.253.91.60)]
    │ nginx 리버스 프록시
    │ coturn TURN 서버 (3478)
    ▼
[ZeroTier 네트워크 10.235.62.x]
    │
    ├── [플랫폼 서버 192.168.33.33]
    │       FastAPI / Next.js / PostgreSQL
    │       Redis / InfluxDB / Mosquitto
    │       MinIO / MLflow
    │
    └── [엣지 디바이스 10.235.62.222]
            edge-settings (설정/MQTT)
            edge-ptz (PTZ API)
            mediamtx (스트리밍)
            gst+Hailo (AI 처리)

[로컬 망 디바이스 192.168.33.x]
    └── MK3-0f5d86bc (줌 카메라, ZeroTier 없음)
```

---

## 플랫폼 서비스 목록

| 서비스 | 포트 | 역할 |
|--------|------|------|
| FastAPI backend | 8080→8000 | REST API, MQTT consumer, WHEP 프록시 |
| Next.js frontend | 3000 | 웹 UI |
| PostgreSQL 15 | 5432 | 디바이스/사용자/설정 영구 저장 |
| Redis | 6379 | 디바이스 온라인 상태(TTL 120초), PTZ 캐시(TTL 30초) |
| InfluxDB 2.7 | 8086 | 시계열 로그 |
| Mosquitto | 1883, 9883(WS) | MQTT 브로커 |
| MinIO | 9000, 9001 | 펌웨어/모델 스토리지 |
| MLflow | 5000 | 모델 실험 추적 |
| Grafana | 3001 | 대시보드 |

---

## 엣지 서비스 목록

| 서비스 | 포트 | 역할 |
|--------|------|------|
| edge-settings | 80 | 메인 HTTP API, MQTT 구독, 설정 관리 |
| edge-ptz | 7777 | PTZ API, ONVIF 클라이언트 |
| mediamtx | 8554(RTSP), 8889(WebRTC), 8888(HLS) | 스트리밍 서버 |
| edge-healthcheck | — | 헬스체크 |
| onvif_srvd | 1000 | ONVIF 서버 |

관련 문서: [[networking]], [[webrtc-streaming]], [[mqtt-communication]]
