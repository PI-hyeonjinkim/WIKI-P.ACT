# 펌웨어 배포 절차

> 출처: raw/decisions/2026-04-17_operations.md
> 마지막 갱신: 2026-04-17

---

## 개요

엣지 소프트웨어(`edge/` 폴더)를 tarball로 빌드 → MinIO 업로드 → 장비에서 install.sh 실행.

---

## 1단계: 빌드 (로컬 → MinIO)

```
/firmware-update build
```

- `edge/` 폴더의 파일들을 tarball로 묶어 MinIO 업로드
- 업로드 경로: `firmware/edge-sw/latest/edge-sw.tar.gz`
- install.sh: `firmware/edge-sw/install.sh`
- 버전: `firmware/edge-sw/latest/version.txt`

**포함 파일 목록:**
```
device_id.py, device_register.py, event_dispatcher.py,
generate_device_id.sh, health_check.py, install.sh,
intelligent_tracker.py, mqtt_logger.py, people_counter.py,
pid_controller.py, ptz_api.py, redis_config_wide.json,
mediamtx.yml, settings_handler.py, wide_detection_fixed.sh,
yolov8_config.json,
systemd/edge-*.service, systemd/mediamtx.service,
systemd/onvif_srvd.service, systemd/wsdd.service
```

---

## 2단계: 배포 (MinIO → 장비)

### 로컬 망 장비 (.66)
```
/firmware-update deploy .66
```

### ZeroTier 장비 (.222, P.ACT01)
```
/firmware-update build deploy .222
```
※ .222는 .33 서버를 통해 SSH 점프.

### 수동 배포 (장비 직접 접속 후)
```bash
curl -fsSL https://api.paidevteam.com/firmware/edge-sw/install.sh | bash
```

---

## mediamtx 설정 백업/롤백

배포 전 중요 설정 변경 시 반드시 백업:
```bash
sudo cp /opt/mediamtx/mediamtx.yml /opt/mediamtx/mediamtx.yml.bak
```

롤백:
```bash
sudo cp /opt/mediamtx/mediamtx.yml.bak /opt/mediamtx/mediamtx.yml
sudo systemctl restart mediamtx
```
