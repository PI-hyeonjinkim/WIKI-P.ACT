# ONVIF 서버 (onvif_srvd)

> 출처: docs/edge/onvif.md, memory/onvif-ghost-service.md (2026-04-22)
> 마지막 갱신: 2026-04-22

---

## 개요

MK3pro에서 동작하는 ONVIF Profile S 호환 C++ 서버.  
NVR/VMS가 카메라를 표준 ONVIF로 발견/제어할 수 있게 한다.

**포트**: 1000 (SOAP) | **systemd**: `onvif_srvd.service`

---

## 아키텍처

```
NVR / VMS
    │ ONVIF SOAP (포트 1000)
    ▼
onvif_srvd
    ├──▶ /dev/ttyAMA0  (Pan 모터)
    └──▶ /dev/ttyAMA2  (Tilt 모터)
```

`ptz_api.py`는 ONVIF 클라이언트로서 onvif_srvd에 SOAP 요청을 보낸다.

---

## 주요 이슈: Ghost Service (카메라 혼자 움직임)

**증상**: 아무도 제어 안 하는데 카메라가 혼자 움직임  
**원인**: 구버전 `onvif.service`가 `/etc/systemd/system/`에 잔존하여 충돌  
**해결**:
```bash
sudo systemctl disable --now onvif.service
sudo systemctl enable --now onvif_srvd.service
```

> 상세: [[onvif-ghost-service]]

---

## 지원 범위

- Profile S (RTSP URL 제공)
- PTZ (Continuous Move, Stop, Presets)
- WS-Discovery (wsdd와 연동)
