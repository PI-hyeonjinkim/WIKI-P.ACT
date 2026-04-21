# 컴파일 로그

> 이 파일은 Claude가 raw/ 를 처리할 때마다 자동으로 기록합니다.

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
