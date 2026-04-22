# 엣지 SW — Python 서비스 및 운영

> 출처: memory/edge-device.md, operations/edge-deployment.md, docs/edge/ (2026-04-22)
> 마지막 갱신: 2026-04-22

---

## 접속 정보

| 항목 | .66 | .45 |
|------|-----|-----|
| Host | 192.168.33.66 | 192.168.33.45 |
| User / Pass | ptz / ptz | ptz / ptz |
| Device ID | MK3-0f5d86bc | MK3-005f607f |
| HW | RPi CM5 + Hailo-8L | RPi CM5 + Hailo-8L |

---

## 서비스 및 포트

| 서비스 (systemd) | 포트 | 설명 |
|------------------|------|------|
| `mediamtx` | 8554 (RTSP), 8888 (HLS/WebRTC) | 미디어 서버 |
| `edge-settings` | 7778 | 설정 API + MQTT 동기화 |
| `edge-ptz` | 7777 | PTZ REST API → onvif_srvd SOAP |
| `edge-healthcheck` | — | 30초마다 헬스체크 MQTT 전송 |
| `edge-device-id` | — | 부팅 시 플랫폼 자동 등록 |
| `onvif_srvd` | 1000 | ONVIF SOAP 서버 |
| `wsdd` | 3702 (UDP) | WS-Discovery |

---

## Python 파일 목록

| 파일 | 역할 |
|------|------|
| `settings_handler.py` | 설정 웹서버 + MediaMTX API + MQTT 동기화 |
| `ptz_api.py` | PTZ REST API → onvif_srvd SOAP 변환 |
| `people_counter.py` | Hailo NPU 피플카운팅 파이프라인 |
| `device_register.py` | 부팅 시 플랫폼 자동 등록 |
| `health_check.py` | 헬스체크 루프 |
| `intelligent_tracker.py` | PID closed-loop 자동 추적 |
| `mqtt_logger.py` | logging → MQTT 핸들러 |
| `device_id.py` | DEVICE_ID 생성/조회 (CPU Serial 기반) |
| `event_dispatcher.py` | 이벤트 → MQTT/Webhook 발행 |

---

## DEVICE_ID 규칙

- 우선순위: 환경변수 `DEVICE_ID` > `/proc/cpuinfo Serial` > `MK3-{hostname}`
- 형식: `MK3-{CPU Serial 뒤 8자리}` (예: `MK3-0f5d86bc`)

---

## 부팅 자동 등록 흐름

```
전원 ON → device_register.py
  → POST https://api.paidevteam.com/api/devices/register
  → 플랫폼 DB에 owner_id=NULL로 등록
  → 사용자가 웹 UI에서 Claim
```

---

## AI ON/OFF 전환 (settings_handler)

```
AI OFF: rpiCamera 활성화 → MediaMTX API로 카메라 설정 가능
AI ON:  rpiCamera 비활성화 → rpicam-vid 파이프 → Hailo NPU → RTSP 출력
```

---

## 모터 시리얼 프로토콜

- Pan: `/dev/ttyAMA0`, Tilt: `/dev/ttyAMA2`, 115200 baud
- 명령: `S`(시작), `T{radian}`(목표위치), `R`(리셋), `V`(버전)

| 설치 모드 | pan_motor_sign | tilt_motor_sign |
|-----------|---------------|-----------------|
| stand (정립) | -1.0 | 1.0 |
| hang (천장) | 1.0 | -1.0 |

`settings.json → ptz.mount_mode`

---

## 배포

```bash
# SCP 단일 파일
scp edge/settings_handler.py ptz@192.168.33.66:/home/ptz/edge/

# 재시작
ssh ptz@192.168.33.66 "sudo systemctl restart edge-settings"
```

Claude Code에서: `"엣지 배포해줘"` (deploy-orchestrator 스킬 자동 실행)

---

## 서비스 재시작 및 진단

```bash
# 재시작
sudo systemctl restart edge-settings edge-ptz mediamtx

# 로그 확인
journalctl -u edge-settings -n 50 --no-pager
journalctl -u edge-ptz -n 50 --no-pager

# 부팅 후 체크리스트
sudo ss -tnp state established | grep 1883   # MQTT 연결
systemctl is-active edge-ptz edge-settings mediamtx
curl -s http://localhost:9997/v3/paths/list  # 스트림 상태

# 설정 확인
cat /home/ptz/edge/settings.json
```

> ⚠️ journald 재부팅 후 로그 소실 방지:
> `sudo sed -i 's/^#*Storage=.*/Storage=persistent/' /etc/systemd/journald.conf && sudo systemctl restart systemd-journald`
