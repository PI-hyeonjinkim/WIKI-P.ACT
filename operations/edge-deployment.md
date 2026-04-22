# 엣지 디바이스 배포 및 운영 가이드

> 출처: memory/edge-device.md (2026-03-14), docs/edge/ (2026-04-22)
> 마지막 갱신: 2026-04-22

---

## 접속 정보

| 항목 | .66 | .45 |
|------|-----|-----|
| Host | 192.168.33.66 | 192.168.33.45 |
| User | ptz | ptz |
| Pass | ptz | ptz |
| Device ID | MK3-0f5d86bc | MK3-005f607f |
| HW | RPi CM5 + Hailo-8L | RPi CM5 + Hailo-8L |
| 홈 디렉터리 | /home/ptz | /home/ptz |

---

## 실행 중인 서비스 및 포트

| 서비스 (systemd) | 포트 | 설명 |
|------------------|------|------|
| `mediamtx` | 8554 (RTSP), 8888 (HLS/WebRTC) | 미디어 서버 |
| `edge-settings` | 7778 | 설정 API + MQTT 동기화 |
| `edge-ptz` | 7777 | PTZ REST API (ONVIF 클라이언트) |
| `edge-healthcheck` | — | 30초마다 헬스체크 MQTT 전송 |
| `edge-device-id` | — | 부팅 시 플랫폼 자동 등록 |
| `onvif_srvd` | 1000 | ONVIF SOAP 서버 |
| `wsdd` | 3702 (UDP 멀티캐스트) | WS-Discovery (NVR 자동 발견) |

Device UI (Next.js 정적): `/home/ptz/device-ui/` → `edge-settings`가 서빙

---

## edge/ 파일 목록

| 파일 | 용도 |
|------|------|
| `settings_handler.py` | 설정 웹서버 (포트 7778) + MQTT 동기화 |
| `ptz_api.py` | PTZ REST API (포트 7777) → onvif_srvd SOAP |
| `people_counter.py` | Hailo NPU 피플카운팅 AI 파이프라인 |
| `device_register.py` | 부팅 시 플랫폼 자동 등록 |
| `health_check.py` | 헬스체크 루프 |
| `intelligent_tracker.py` | PID closed-loop 자동 추적 |
| `mqtt_logger.py` | Python logging → MQTT 핸들러 |
| `device_id.py` | DEVICE_ID 생성/조회 (CPU Serial 기반) |
| `event_dispatcher.py` | 이벤트 → MQTT/Webhook 발행 |
| `mediamtx.yml` | MediaMTX 설정 |
| `install.sh` | 신규 장비 초기 설치 스크립트 |

---

## DEVICE_ID 규칙

- 우선순위: 환경변수 `DEVICE_ID` > `/proc/cpuinfo Serial` > `MK3-{hostname}`
- 형식: `MK3-{CPU Serial 뒤 8자리}` (예: `MK3-0f5d86bc`)

---

## 자동 등록 흐름

```
라즈베리파이 전원 ON
  → device_register.py 실행
  → POST https://api.paidevteam.com/api/devices/register
  → CPU Serial 기반 device_id 생성
  → 플랫폼 DB에 owner_id=NULL로 등록
  → 사용자가 웹 UI에서 Claim 버튼으로 할당
```

---

## 파일 배포 방법

```bash
# SCP 업로드
scp edge/settings_handler.py ptz@192.168.33.66:/home/ptz/edge/

# SSH로 서비스 재시작
ssh ptz@192.168.33.66 "sudo systemctl restart edge-settings"

# 서비스 상태 확인
ssh ptz@192.168.33.66 "systemctl status edge-settings edge-ptz mediamtx onvif_srvd"
```

Claude Code에서는 `deploy-edge` 스킬 또는 `"엣지 배포해줘"` 사용.

---

## Device UI 배포

```bash
# 빌드 (로컬)
cd device-ui && npm run build

# 배포 (SCP)
scp -r device-ui/out/ ptz@192.168.33.66:/home/ptz/device-ui/
```

---

## 모터 시리얼 프로토콜 (v1.0.6)

- Pan: `/dev/ttyAMA0`, Tilt: `/dev/ttyAMA2`, 115200 baud
- 명령: `S`(시작), `T{radian}`(목표위치), `R`(리셋), `V`(버전), `C`(캘리브레이션)

### 설치 모드 (mount_mode)

| 모드 | pan_motor_sign | tilt_motor_sign |
|------|---------------|-----------------|
| stand (정립) | -1.0 | 1.0 |
| hang (천장) | 1.0 | -1.0 |

설정: `settings.json → ptz.mount_mode`

---

## AI ON/OFF 전환 (settings_handler)

```
AI OFF: rpiCamera 활성화 → MediaMTX API로 카메라 설정 변경 가능
AI ON:  rpiCamera 비활성화 → rpicam-vid 파이프 → Hailo NPU → RTSP 출력
```

---

## 신규 장비 설치

```bash
# 장비에서 실행
curl -fsSL https://api.paidevteam.com/firmware/install.sh | bash
```

설치 항목: Python 패키지, Hailo Runtime, systemd 서비스, `.hef` 모델 다운로드, device ID 생성
