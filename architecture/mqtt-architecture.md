# MQTT 아키텍처

## 핵심 구조

```
엣지 디바이스
  └─→ paidevteam.com:1883
        └─→ EC2 nginx (TCP 프록시)
              └─→ 10.235.62.211:1883 (ZeroTier VPN)
                    └─→ 192.168.33.33:1883 (local mosquitto)
                          └─→ 백엔드 MQTT Consumer
```

`paidevteam.com:1883`은 **별도 MQTT 브로커가 아님**. EC2의 nginx가 TCP 프록시로 로컬 mosquitto(192.168.33.33:1883)를 포워딩하는 구조.

## 절대 하지 말아야 할 것

**mosquitto.conf에 paidevteam.com 브리지를 추가하면 안 됨.**

→ 자기 자신에게 루프가 생겨 PTZ 명령이 초당 252개로 폭발함 (실제 사고 경험).

엣지가 `paidevteam.com:1883`에 구독 = 실제로는 local mosquitto에 직접 연결되는 것과 동일. 브리지 불필요.

## MQTT 토픽 구조

```
logs/{device_id}/app          — QoS 1  앱 로그
logs/{device_id}/error        — QoS 2  에러 로그
logs/{device_id}/system       — QoS 0  시스템 로그
logs/{device_id}/status       — QoS 0  연결 상태

webrtc/{device_id}/offer      — QoS 1  WebRTC 시그널링
webrtc/{device_id}/answer     — QoS 1
webrtc/{device_id}/hangup     — QoS 1

commands/{device_id}/ptz      — QoS 1  PTZ 제어
commands/{device_id}/settings — QoS 1  설정 변경
commands/{device_id}/ai       — QoS 1  AI 토글
commands/{device_id}/tracking — QoS 1  트래킹 토글

settings/{device_id}/current  — QoS 1  현재 설정 보고
settings/{device_id}/ack      — QoS 1  설정 적용 확인
```

## MQTT 브로커 (mosquitto) 설정

- 위치: `192.168.33.33:1883` (Docker: platform-mosquitto)
- WebSocket: `9883`
- 설정 파일: `mosquitto/config/mosquitto.conf`

> 출처: memory/mqtt-architecture.md (2026-04-08)
