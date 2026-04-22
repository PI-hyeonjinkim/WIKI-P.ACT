# 컴파일 로그

> 이 파일은 Claude가 raw/ 를 처리할 때마다 자동으로 기록합니다.

---

### 2026-04-22 — docs/ 체계 수립 반영

처리한 파일: `docs/edge/`, `docs/platform/`, `docs/aws-proxy/`

갱신된 wiki 항목:
- `wiki/operations/edge-deployment.md` — Device UI 배포, 서비스 포트 정리, AI ON/OFF 전환 로직 추가
- `wiki/architecture/platform-overview.md` — docker-compose 전체 서비스 구성, 백엔드 API 전체 목록, InfluxDB 버킷 추가

변경 요약: docs/ 3개 폴더(edge/platform/aws-proxy) 신설 및 CLAUDE.md 재정리에 따라 위키 관련 항목 병합 갱신.

---

### 2026-04-22 — memory/ 인계 문서 위키 이전

출처: Claude Code 프로젝트 메모리 (`~/.claude/projects/.../memory/`)

처리한 메모리 파일:
- `memory/onvif-ghost-service.md`
- `memory/edge-device.md`
- `memory/mqtt-architecture.md`
- `memory/platform.md`
- `memory/mokpo-deploy.md`

생성된 wiki 항목:
- `wiki/troubleshooting/onvif-ghost-service.md` (신규) — 카메라 혼자 움직이는 문제
- `wiki/architecture/mqtt-architecture.md` (신규) — MQTT 브로커 구조 + 토픽 목록
- `wiki/architecture/platform-overview.md` (신규) — FastAPI 백엔드 API + 인증 구조
- `wiki/operations/edge-deployment.md` (신규) — 엣지 디바이스 배포 가이드
- `wiki/operations/mokpo-field.md` (신규) — 목포 물류장 운영 가이드

변경 요약: 팀 인계용 핵심 운영 지식 5건 위키에 수록. 트러블슈팅 1건 (ONVIF ghost service), 아키텍처 2건 (MQTT, 플랫폼), 운영 절차 2건 (엣지 배포, 목포 현장).

---

## 형식

```
### YYYY-MM-DD
- 처리한 파일: raw/폴더/파일명.md
- 생성/갱신된 wiki 항목: wiki/폴더/파일명.md
- 변경 요약: 한 줄 설명
```

---

## 이력

### 2026-04-21
- 처리한 파일: raw/troubleshooting/2026-04-21_ec2-nginx-websocket-mqtt-tunnel.md
- 생성된 wiki 항목: wiki/troubleshooting/ec2-nginx-websocket.md
- 변경 요약: EC2 nginx WebSocket 경로 누락 + http2 충돌로 영상/MQTT 불가 트러블슈팅 추가

### 2026-04-17 — 초기화
- 위키 구조 생성 (raw/, wiki/, output/ 폴더 및 CLAUDE.md)
- wiki/index.md 초기화, wiki/log.md 초기화
- 처리된 raw 파일: 없음

### 2026-04-17 — 초기 컴파일

처리한 raw 파일:
- `raw/decisions/2026-04-17_platform-architecture.md`
- `raw/decisions/2026-04-17_operations.md`
- `raw/devices/2026-04-17_device-inventory.md`
- `raw/troubleshooting/2026-04-17_known-issues.md`

생성된 wiki 항목:
- `wiki/architecture/system-overview.md` (신규)
- `wiki/architecture/networking.md` (신규)
- `wiki/architecture/webrtc-streaming.md` (신규)
- `wiki/troubleshooting/webrtc-blackscreen.md` (신규, 케이스 A/B/C)
- `wiki/troubleshooting/mqtt-reconnect.md` (신규)
- `wiki/devices/inventory.md` (신규)
- `wiki/operations/firmware-update.md` (신규)
- `wiki/operations/service-management.md` (신규)

변경 요약: 플랫폼 아키텍처, 네트워킹, WebRTC 파이프라인, 트러블슈팅 5건, 디바이스 인벤토리, 운영 절차 초기 수록.
