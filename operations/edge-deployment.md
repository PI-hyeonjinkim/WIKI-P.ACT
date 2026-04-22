# 엣지 디바이스 배포 및 운영 가이드

## 접속 정보

| 항목 | 값 |
|------|-----|
| Host | 192.168.33.66 |
| User | ptz |
| Pass | ptz |
| 홈 디렉토리 | /home/ptz |
| 배포 경로 | /home/ptz/edge/ |

## 디바이스 현황

| IP | Device ID | HW | 용도 |
|----|-----------|----|----|
| 192.168.33.49 | MK3-b630b31e | RPi5 | 테스트 |
| 192.168.33.66 | MK3-0f5d86bc | RPi CM5 + Hailo-8L | 메인 테스트 (듀얼카메라, PTZ) |
| 192.168.33.28 | — | RPi5 | 운영 (수정 금지) |

## 실행 중인 서비스

| 서비스 | 포트 | 설명 |
|--------|------|------|
| mediamtx | 8554/8889/8888 | RTSP/WebRTC/HLS 미디어 서버 |
| edge-settings | 80 | 설정 API + Device UI + MQTT 동기화 |
| edge-ptz | 7777 | PTZ REST API (시리얼 + ONVIF 줌) |
| edge-healthcheck | — | 60초 헬스체크 MQTT 전송 |
| edge-whep-bridge | — | MQTT↔WHEP WebRTC 시그널링 |
| onvif_srvd | 1000 | ONVIF SOAP 서버 |
| wsdd | 3702 | WS-Discovery |
| redis | 6379 | AI 감지 결과 PubSub |

## edge/ 폴더 파일 목록

| 파일 | 용도 |
|------|------|
| `device_id.py` | DEVICE_ID 공통 모듈 (CPU Serial 자동 생성) |
| `device_register.py` | 부팅 시 플랫폼 자동 등록 |
| `mqtt_logger.py` | Python logging → MQTT 핸들러 |
| `health_check.py` | 60초마다 CPU/메모리/온도 전송 |
| `settings_handler.py` | 설정 API 서버 + MQTT 동기화 |
| `ptz_api.py` | PTZ REST API (시리얼 + ONVIF 줌) |
| `intelligent_tracker.py` | PID closed-loop PTZ 자동 추적 |
| `people_counter.py` | People Counting AI |
| `install.sh` | 신규 장비 초기 설치 스크립트 |
| `mediamtx.yml` | MediaMTX 미디어 서버 설정 |

## DEVICE_ID 규칙

- 우선순위: 환경변수 DEVICE_ID > /proc/cpuinfo Serial > `MK3-{hostname}`
- 형식: `MK3-{CPU Serial 뒤 8자리}` (예: `MK3-0f5d86bc`)

## 자동 등록 흐름

```
라즈베리파이 전원 ON
  → device_register.py 실행
  → POST https://api.paidevteam.com/api/devices/register
  → CPU Serial 기반 device_id 생성
  → 플랫폼 DB에 owner_id=NULL로 등록
  → 사용자가 웹 UI에서 Claim 버튼으로 할당
```

## 모터 시리얼 프로토콜 (v1.0.6)

- Pan: `/dev/ttyAMA0`, Tilt: `/dev/ttyAMA2`, 115200 baud
- 명령: `S`(시작), `T{radian}`(목표위치), `R`(리셋), `V`(버전), `C`(캘리브레이션)
- T 명령 응답: `Target angle` / `Current angle` (실제 위치, closed-loop 피드백 가능) / `Sensor offset`

## 설치 모드 (mount_mode)

| 모드 | pan_motor_sign | tilt_motor_sign |
|------|---------------|-----------------|
| stand (정립) | -1.0 | 1.0 |
| hang (천장) | 1.0 | -1.0 |

설정: `settings.json → ptz.mount_mode`

## 파일 배포 방법

```bash
# SCP 업로드
scp edge/settings_handler.py ptz@192.168.33.66:/home/ptz/edge/

# SSH로 서비스 재시작
ssh ptz@192.168.33.66 "sudo systemctl restart edge-settings"

# 서비스 상태 확인
ssh ptz@192.168.33.66 "systemctl status edge-settings"
```

Claude Code에서는 `deploy-edge` 스킬 사용.

> 출처: memory/edge-device.md (2026-03-14)
