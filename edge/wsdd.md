# WS-Discovery (wsdd)

> 출처: docs/edge/wsdd.md (2026-04-22)
> 마지막 갱신: 2026-04-22

---

## 개요

NVR의 "카메라 검색"에 응답하는 WS-Discovery 데몬.

**systemd**: `wsdd.service` | **프로토콜**: UDP 멀티캐스트 239.255.255.250:3702

---

## 동작

```
NVR (Probe 멀티캐스트 UDP:3702)
    ▼
wsdd (MK3pro)
    │ ProbeMatch 응답
    └──▶ NVR이 카메라 발견 → ONVIF 연결
```

---

## 트러블슈팅

| 증상 | 확인 사항 |
|------|-----------|
| NVR에서 발견 안 됨 | `systemctl status wsdd` / UDP 3702 방화벽 |
| 잘못된 IP로 발견 | `wsdd_start.sh`의 `--interface` 옵션 확인 |
