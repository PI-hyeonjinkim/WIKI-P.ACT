# P.ACT Wiki — 전체 목차

> 이 파일은 Claude가 자동 관리합니다. 직접 편집하지 마세요.
> 마지막 갱신: 2026-04-22

---

## 엣지 디바이스 (`edge/`)

- [[edge-sw]] — Python 서비스 (settings_handler, ptz_api, people_counter), 배포, 진단
- [[device-ui]] — 카메라 설정 웹 UI (Next.js 정적 빌드, 배포 방법)
- [[onvif]] — ONVIF C++ 서버 (onvif_srvd), Ghost Service 이슈
- [[wsdd]] — WS-Discovery 데몬 (NVR 카메라 자동 발견)
- [[firmware-update]] — 펌웨어 빌드·배포·롤백 절차
- [[mokpo-field]] — 목포 물류장 14대 배포 현황, 운영 이슈

---

## 플랫폼 서버 (`platform/`)

- [[overview]] — 전체 구조도, 플랫폼 서비스 목록, 서버 관리
- [[backend]] — FastAPI API 목록, 인증 구조, DB 스키마, InfluxDB 버킷
- [[webrtc]] — WebRTC 파이프라인, WHEP 시그널링, mediamtx 경로 설정
- [[mqtt]] — MQTT 브로커 구조, 토픽 목록, 브리지 금지 이유

---

## EC2 프록시 (`aws-proxy/`)

- [[networking]] — ZeroTier, nginx 서브도메인 라우팅, Route53 DNS, TURN 서버

---

## 트러블슈팅 (`troubleshooting/`)

- [[webrtc-blackscreen]] — black screen 3가지 케이스 (mediamtx 충돌 / coturn / WHEP timeout)
- [[mqtt-reconnect]] — 부팅 후 MQTT 미연결, PTZ 명령 미동작
- [[ec2-nginx-websocket]] — EC2 nginx WebSocket 미설정으로 영상/MQTT 연결 불가
- [[onvif-ghost-service]] — 카메라 혼자 움직이는 원인: 구버전 onvif.service 잔존

---

## 디바이스 인벤토리 (`devices/`)

- [[inventory]] — 플랫폼 서버, EC2, MK3-0f5d86bc, MK3-005f607f(P.ACT01), 목포 물류장
