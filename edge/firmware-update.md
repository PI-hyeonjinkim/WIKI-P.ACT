# 펌웨어 배포 절차

> 출처: raw/decisions/2026-04-17_operations.md
> 마지막 갱신: 2026-04-17

---

## 개요

`edge/` 폴더를 tarball로 빌드 → MinIO 업로드 → 장비에서 install.sh 실행.

---

## 1단계: 빌드 (로컬 → MinIO)

```
/firmware-update build
```

업로드 경로:
- `firmware/edge-sw/latest/edge-sw.tar.gz`
- `firmware/edge-sw/install.sh`
- `firmware/edge-sw/latest/version.txt`

**포함 파일:**
```
device_id.py, device_register.py, event_dispatcher.py,
health_check.py, install.sh, intelligent_tracker.py,
mqtt_logger.py, people_counter.py, pid_controller.py,
ptz_api.py, settings_handler.py, mediamtx.yml,
systemd/edge-*.service, systemd/mediamtx.service,
systemd/onvif_srvd.service, systemd/wsdd.service
```

---

## 2단계: 배포

### 로컬 망 장비 (.66, .45)
```
/firmware-update deploy .66
```

### ZeroTier 장비 (P.ACT01 등)
```
/firmware-update build deploy .222
```
※ ZeroTier 장비는 .33 서버를 통해 SSH 점프.

### 수동 배포
```bash
# 장비에서 직접 실행
curl -fsSL https://api.paidevteam.com/firmware/edge-sw/install.sh | bash
```

---

## mediamtx 설정 롤백

```bash
# 배포 전 백업
sudo cp /opt/mediamtx/mediamtx.yml /opt/mediamtx/mediamtx.yml.bak

# 롤백
sudo cp /opt/mediamtx/mediamtx.yml.bak /opt/mediamtx/mediamtx.yml
sudo systemctl restart mediamtx
```
