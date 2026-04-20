# 네트워킹 구조

> 출처: raw/decisions/2026-04-17_platform-architecture.md
> 마지막 갱신: 2026-04-17

---

## ZeroTier

원격 엣지 디바이스와 플랫폼을 연결하는 사설 overlay 네트워크.
- 네트워크 대역: 10.235.62.x
- 주요 노드: EC2(10.235.62.214), P.ACT01(10.235.62.222)
- 로컬 디바이스(.66)는 ZeroTier 미사용 → 직접 접근 불가 (로컬 망만)

---

## EC2 프록시 (api.paidevteam.com)

| 역할 | 설정 |
|------|------|
| HTTPS | nginx → ZeroTier → 192.168.33.33:8080 |
| MQTT TCP | nginx TCP 프록시 1883 → mosquitto |
| MQTT WS | nginx → /mqtt → mosquitto 9883 |
| TURN | coturn 3478 (UDP/TCP) |
| 펌웨어 | nginx → /firmware → MinIO |

---

## TURN 서버 (coturn)

WebRTC에서 브라우저가 ZeroTier IP 디바이스에 직접 연결 불가한 경우 릴레이.

**설정 파일**: /etc/turnserver.conf (api.paidevteam.com)
```
listening-port=3478
realm=api.paidevteam.com
use-auth-secret
static-auth-secret=pact_turn_2026
external-ip=54.253.91.60/172.31.36.71   ← EC2 NAT 필수
min-port=49152
max-port=65535
```

**인증 방식**: HMAC-SHA1 time-limited credentials
- 백엔드: `TURN_SECRET=pact_turn_2026`으로 생성
- mediamtx: `username: AUTH_SECRET, password: pact_turn_2026` (자동 HMAC 생성)

> ⚠️ **주의**: `external-ip` 없으면 릴레이 주소가 사설 IP로 광고되어 외부 연결 불가
> 상세: [[webrtc-blackscreen-coturn]]

---

## 브라우저 → 디바이스 연결 경로

| 디바이스 위치 | 브라우저 위치 | ICE 경로 |
|--------------|--------------|----------|
| 로컬 망(192.168.33.x) | 같은 로컬 망 | 직접 연결 (TURN 불필요) |
| ZeroTier(10.235.62.x) | 외부 | TURN 릴레이 필수 |
| ZeroTier(10.235.62.x) | ZeroTier 내부 | 직접 연결 가능 |

---

## MQTT 브리지 금지 사항

> ⚠️ paidevteam.com:1883은 nginx TCP 프록시 → 내부 mosquitto
> 브리지 추가 시 mosquitto → nginx → mosquitto 루프 발생
> 브리지 설정 절대 금지
