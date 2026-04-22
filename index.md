# P.ACT Wiki — 전체 목차

> 이 파일은 Claude가 자동 관리합니다. 직접 편집하지 마세요.
> 마지막 갱신: 2026-04-22

---

## 아키텍처 (`architecture/`)

- [[system-overview]] — 제품 개요, 전체 구조도, 서비스 목록
- [[networking]] — ZeroTier, EC2 프록시, coturn TURN 서버, Route53 DNS, MQTT 주의사항
- [[webrtc-streaming]] — WebRTC 파이프라인, WHEP 시그널링, mediamtx 경로 설정
- [[mqtt-architecture]] — MQTT 브로커 실제 구조, 토픽 목록, 브리지 추가 금지 이유
- [[platform-overview]] — FastAPI 백엔드 API, 인증 구조, docker-compose 서비스 구성, DB 스키마

---

## 트러블슈팅 (`troubleshooting/`)

- [[webrtc-blackscreen]] — black screen 3가지 케이스 (mediamtx 충돌 / coturn / WHEP timeout)
- [[mqtt-reconnect]] — 부팅 후 MQTT 미연결, PTZ 명령 미동작
- [[ec2-nginx-websocket]] — EC2 nginx WebSocket 미설정으로 영상/MQTT 연결 불가 (2026-04-21)
- [[onvif-ghost-service]] — 카메라 혼자 움직이는 원인: 구버전 onvif.service 잔존, 진단·해결 방법

---

## 디바이스 인벤토리 (`devices/`)

- [[inventory]] — 플랫폼 서버, EC2, MK3-0f5d86bc, MK3-005f607f(P.ACT01), 목포 물류장

---

## 운영 절차 (`operations/`)

- [[firmware-update]] — 펌웨어 빌드·배포 절차, 롤백
- [[service-management]] — 서비스 재시작, 로그 확인, 부팅 후 진단 체크리스트
- [[edge-deployment]] — 엣지 디바이스 접속/서비스/배포/Device UI/모터 프로토콜 (2026-04-22 갱신)
- [[mokpo-field]] — 목포 물류장 14대 배포 현황, 운영 이슈, Grafana 접속 정보
